# Configure the WebdriverIO runner

Open when configuring WDIO, building the Tauri app and managing the platform driver.

[Topic context](webdriverio-suite-configuration-execution.md)

Save as `e2e-tests/wdio.conf.js`; use the [project setup](webdriverio-test-project-setup.md) and [test spec](webdriverio-application-spec.md). Set the application basename for your app.

```javascript
import os from 'os';
import path from 'path';
import { spawn, spawnSync } from 'child_process';
import { fileURLToPath } from 'url';
const __dirname = fileURLToPath(new URL('.', import.meta.url));
// keep track of the `tauri-driver` child process
let tauriDriver;
let exit = false;
export const config = {
  hostname: '127.0.0.1',
  port: 4444,
  specs: ['./test/specs/**/*.js'],
  maxInstances: 1,
  capabilities: [
    {
      maxInstances: 1,
      'tauri:options': {
        application: path.resolve(__dirname, '../src-tauri/target/debug',
          process.platform === 'win32' ? 'tauri-app.exe' : 'tauri-app'),
      },
    },
  ],
  reporters: ['spec'],
  framework: 'mocha',
  mochaOpts: {
    ui: 'bdd',
    timeout: 60000,
  },
  // ensure the rust project is built since we expect this binary to exist for the webdriver sessions
  onPrepare: () => {
    // Remove the extra `--` if you're not using npm!
    const build = spawnSync(
      'npm',
      ['run', 'tauri', 'build', '--', '--debug', '--no-bundle'],
      {
        cwd: path.resolve(__dirname, '..'),
        stdio: 'inherit',
        shell: true,
      }
    );
    if (build.error || build.status !== 0) throw build.error ?? new Error(`Build failed: ${build.status}`);
  },
  // ensure we are running `tauri-driver` before the session starts so that we can proxy the webdriver requests
  beforeSession: () => {
    exit = false;
    tauriDriver = spawn(
      path.resolve(os.homedir(), '.cargo', 'bin', process.platform === 'win32' ? 'tauri-driver.exe' : 'tauri-driver'),
      [],
      { stdio: [null, process.stdout, process.stderr] }
    );
    tauriDriver.on('error', (error) => {
      console.error('tauri-driver error:', error);
      process.exit(1);
    });
    tauriDriver.on('exit', (code) => {
      if (!exit) {
        console.error('tauri-driver exited with code:', code);
        process.exit(1);
      }
    });
  },
  // clean up the `tauri-driver` process we spawned at the start of the session
  // note that afterSession might not run if the session fails to start, so we also run the cleanup on shutdown
  afterSession: () => {
    closeTauriDriver();
  },
};
function closeTauriDriver() {
  exit = true;
  tauriDriver?.kill();
}
function onShutdown(fn) {
  const cleanup = () => {
    try {
      fn();
    } finally {
      process.exit();
    }
  };
  process.on('exit', cleanup);
  process.on('SIGINT', cleanup);
  process.on('SIGTERM', cleanup);
  process.on('SIGHUP', cleanup);
  process.on('SIGBREAK', cleanup);
}
// ensure tauri-driver is closed when our test process exits
onShutdown(() => {
  closeTauriDriver();
});
```
