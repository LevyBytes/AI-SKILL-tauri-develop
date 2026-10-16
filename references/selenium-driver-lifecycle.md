# Selenium driver lifecycle

Open when building the app, starting tauri-driver and cleaning up the Selenium session.

[Topic context](selenium-tests-driver-lifecycle.md)

Save this complete module as `e2e-tests/test/driver.js`. Set the application basename to match your project. Use it with [the test module](selenium-app-assertions.md) and the [project setup](selenium-test-project-setup.md). The driver must remain alive until `driver.quit()` finishes.

```javascript
import os from 'node:os';
import path from 'node:path';
import { spawn, spawnSync } from 'node:child_process';
import { fileURLToPath } from 'node:url';
import { Builder, Capabilities } from 'selenium-webdriver';

const root = fileURLToPath(new URL('../../', import.meta.url));
const suffix = process.platform === 'win32' ? '.exe' : '';
const application = path.join(root, 'src-tauri', 'target', 'debug', `tauri-app${suffix}`);
export let driver;
let tauriDriver;
let stopping = false;
let closing;

export async function startDriver() {
  // Build before attempting to connect to the driver.
  const build = spawnSync('yarn', ['tauri', 'build', '--debug', '--no-bundle'], {
    cwd: root, stdio: 'inherit', shell: true,
  });
  if (build.error || build.status !== 0) {
    throw build.error ?? new Error(`Tauri build failed: ${build.status}`);
  }
  tauriDriver = spawn(
    path.join(os.homedir(), '.cargo', 'bin', `tauri-driver${suffix}`),
    [], { stdio: ['ignore', process.stdout, process.stderr] }
  );
  tauriDriver.on('error', (error) => {
    console.error('tauri-driver error:', error);
    process.exit(1);
  });
  tauriDriver.on('exit', (code) => {
    if (!stopping) {
      console.error('tauri-driver exited unexpectedly:', code);
      process.exit(1);
    }
  });
  const capabilities = new Capabilities();
  capabilities.set('tauri:options', { application });
  capabilities.setBrowserName('wry');
  try {
    driver = await new Builder().withCapabilities(capabilities)
      .usingServer('http://127.0.0.1:4444/').build();
  } catch (error) {
    await stopDriver();
    throw error;
  }
}

export function stopDriver() {
  // Share one cleanup promise, including when setup failed partway through.
  if (!closing) {
    stopping = true;
    closing = (async () => {
      try {
        await driver?.quit();
      } finally {
        tauriDriver?.kill();
      }
    })();
  }
  return closing;
}

// An exit handler can do synchronous cleanup only.
process.on('exit', () => { stopping = true; tauriDriver?.kill(); });
for (const signal of ['SIGINT', 'SIGTERM', 'SIGHUP', 'SIGBREAK']) {
  process.once(signal, () => {
    const deadline = setTimeout(() => process.exit(1), 5000);
    stopDriver().catch(console.error).finally(() => {
      clearTimeout(deadline);
      process.exit(1);
    });
  });
}
```
