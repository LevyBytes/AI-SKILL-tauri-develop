# Ci, Example

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: WebDriver; Continuous Integration; Selenium; WebdriverIO.

## WebDriver

### WebDriver

[WebDriver](https://www.w3.org/TR/webdriver/) defines a common protocol for driving web pages, built mainly for test automation. Tauri exposes this [WebDriver](https://www.w3.org/TR/webdriver/) protocol through [`tauri-driver`](https://crates.io/crates/tauri-driver), a portable wrapper that delegates to whatever native [WebDriver](https://www.w3.org/TR/webdriver/) server each platform ships. For desktop targets, coverage is limited to Windows and Linux, since macOS lacks a usable WKWebView driver. Appium 2 enables iOS and Android, though that path has yet to be made smooth.

### System Dependencies

Grab a fresh copy of [`tauri-driver`](https://crates.io/crates/tauri-driver), or bring a copy you already have up to date, with:

Terminal window

```
cargo install tauri-driver --locked
```

Since the tooling relies on each system's built-in [WebDriver](https://www.w3.org/TR/webdriver/) server, every supported platform imposes its own prerequisites before [`tauri-driver`](https://crates.io/crates/tauri-driver) will work.

### Linux

On Linux the driver of choice is `WebKitWebDriver`. To find out whether it is already present, run `which WebKitWebDriver`; certain distributions ship it inside their main WebKit package. Elsewhere it lives in its own package instead, for instance `webkit2gtk-driver` on Debian-derived systems.

### Windows

Be sure the [Microsoft Edge Driver](https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/) you obtain lines up with the Edge release present on the Windows machine where your app gets built and exercised. For a current Windows setup that is nearly always the newest stable build. A mismatch between the pair tends to leave your WebDriver suite stuck as it attempts to establish a connection.

Fetching the right Microsoft Edge Driver can be handled for you by the [msedgedriver-tool](https://github.com/chippers/msedgedriver-tool):

Terminal window

```
cargo install --git https://github.com/chippers/msedgedriver-tool& "$HOME/.cargo/bin/msedgedriver-tool.exe"
```

What you download includes a binary named `msedgedriver.exe`. Because [`tauri-driver`](https://crates.io/crates/tauri-driver) searches `$PATH` for it, either place it somewhere on the path or point [`tauri-driver`](https://crates.io/crates/tauri-driver) at it via the `--native-driver` option. Pulling it down automatically during CI provisioning is worth considering, so the Edge and Edge Driver builds remain aligned on Windows CI runners. Instructions for that may show up here in the future.

### Example Applications

What follows are walkthroughs that take you through building a bare-bones sample app and putting it under WebDriver tests.

Should you rather skip ahead to the end product and study a complete, stripped-down repository that puts the walkthrough into practice, head to [https://github.com/tauri-apps/webdriver-example](https://github.com/tauri-apps/webdriver-example).

[Selenium](https://v2.tauri.app/develop/tests/webdriver/example/selenium/) [WebdriverIO](https://v2.tauri.app/develop/tests/webdriver/example/webdriverio/)

### Continuous Integration (CI)

Each example above already ships its own GitHub Actions CI script, yet the WebDriver CI walkthrough further down is still worth a read, since it goes into the underlying idea in greater depth.

[Continuous Integration (CI)](https://v2.tauri.app/develop/tests/webdriver/ci/)

---

## Continuous Integration

### Continuous Integration

Your CI pipeline can drive [WebDriver](https://www.w3.org/TR/webdriver/) tests through [`tauri-driver`](https://crates.io/crates/tauri-driver). The walkthrough here builds on the [WebdriverIO](https://webdriver.io/) sample we [previously built together](https://v2.tauri.app/develop/tests/webdriver/example/webdriverio/), paired with GitHub Actions.

On Linux these WebDriver runs rely on a simulated display. Certain providers, GitHub Actions among them, will additionally let you exercise WebDriver tests on Windows.

### GitHub Actions

This GitHub Actions setup takes the following for granted:

1. Your Tauri app lives under the `src-tauri` folder.
2. The [WebDriverIO](https://webdriver.io/) runner sits in the `e2e-tests` directory and fires off whenever `yarn test` is invoked from there.
   .github/workflows/webdriver.yml

```
# run this action when the repository is pushed toon: [push]
# the name of our workflowname: WebDriver
jobs:  # a single job named test  test:    # the display name of the test job    name: WebDriverIO Test Runner
    # run on the matrix platform    runs-on: ${{ matrix.platform }}    strategy:      # do not fail other matrix runs if one fails      fail-fast: false      # set all platforms our test should run on      matrix:        platform: [ubuntu-latest, windows-latest]
    # the steps our job runs **in order**    steps:      # checkout the code on the workflow runner      - uses: actions/checkout@v4
      # install system dependencies that Tauri needs to compile on Linux.      # note the extra dependencies for `tauri-driver` to run which are: `webkit2gtk-driver` and `xvfb`      - name: Tauri dependencies        if: matrix.platform == 'ubuntu-latest'        run: |          sudo apt-get update &&          sudo apt-get install -y \          libwebkit2gtk-4.1-dev \          libayatana-appindicator3-dev \          webkit2gtk-driver \          xvfb
      # install a matching Microsoft Edge Driver version using msedgedriver-tool      - name: install msdgedriver (Windows)        if: matrix.platform == 'windows-latest'        run: |          cargo install --git https://github.com/chippers/msedgedriver-tool          & "$HOME/.cargo/bin/msedgedriver-tool.exe"          $PWD.Path >> $env:GITHUB_PATH
      # install latest stable Rust release      - name: Setup rust-toolchain stable        uses: dtolnay/rust-toolchain@stable
      # setup caching for the Rust target folder      - name: Setup Rust cache        uses: Swatinem/rust-cache@v2        with:          workspaces: src-tauri
      # we run our Rust tests before the webdriver tests to avoid testing a broken application      - name: Cargo test        run: cargo test
      # install the latest stable node version at the time of writing      - name: Node 24        uses: actions/setup-node@v4        with:          node-version: 24          cache: 'yarn'
      # install the application Node.js dependencies with Yarn      - name: Yarn install        run: yarn install --frozen-lockfile
      # install the e2e-tests Node.js dependencies with Yarn      - name: Yarn install        run: yarn install --frozen-lockfile        working-directory: e2e-tests
      # install the latest version of `tauri-driver`.      # note: the tauri-driver version is independent of any other Tauri versions      - name: Install tauri-driver        run: cargo install tauri-driver --locked
      # run the WebdriverIO test suite on Linux.      # we run it through `xvfb-run` (the dependency we installed earlier) to have a fake      # display server which allows our application to run headless without any changes to the code      - name: WebdriverIO (Linux)        if: matrix.platform == 'ubuntu-latest'        run: xvfb-run yarn test        working-directory: e2e-tests
      # run the WebdriverIO test suite on Windows.      # in this case we can run the tests directly.      - name: WebdriverIO (Windows)        if: matrix.platform == 'windows-latest'        run: yarn test        working-directory: e2e-tests
```

---

## Selenium

### Selenium

For this WebDriver demonstration we lean on [Selenium](https://selenium.dev/) together with a well-known Node.js test harness. The assumption is that Node.js is on your machine already, plus either `npm` or `yarn`, even though `pnpm` is what the [finished example project](https://github.com/tauri-apps/webdriver-example) happens to use.

### Create a Directory for the Tests

Let's carve out somewhere in the project to hold these tests. This sample nests the folder because additional frameworks come up later on, though in practice a single one is usually all you need. Spin up the folder with `mkdir -p e2e-tests`. From here on, the guide takes for granted that your shell sits within the `e2e-tests` directory.

### Initializing a Selenium Project

To get this suite going we start from a ready-made `package.json`, since the dependencies have already been picked for us and the aim is to present a straightforward, functioning setup. Down at the end of this section sits a folded-away walkthrough covering how to assemble it yourself.

`package.json`:

```
{  "name": "selenium",  "version": "1.0.0",  "private": true,  "type": "module",  "scripts": {    "test": "mocha"  },  "dependencies": {    "chai": "^5.2.1",    "mocha": "^11.7.1",    "selenium-webdriver": "^4.34.0"  }}
```

There's a script wiring up [Mocha](https://mochajs.org/) as the framework and surfacing it under the `test` command. Alongside it sit the various packages the tests depend on: [Mocha](https://mochajs.org/) for the test framework itself, [Chai](https://www.chaijs.com/) supplying assertions, and [`selenium-webdriver`](https://www.npmx.dev/package/selenium-webdriver), the [Selenium](https://selenium.dev/) binding for Node.js.

Expand this to learn how to stand up the project from scratch

To pull in the dependencies starting from nothing, simply issue the command below.

- npm
- yarn

```
npm install mocha chai selenium-webdriver
```

```
yarn add mocha chai selenium-webdriver
```

It's also a good idea to drop a `"test": "mocha"` entry into the `"scripts"` key of `package.json`, which lets you kick off Mocha with nothing more than

- npm
- yarn

```
npm test
```

```
yarn test
```

### Testing

In contrast to the [WebdriverIO Test Suite](https://v2.tauri.app/develop/tests/webdriver/example/webdriverio/#config), Selenium ships without any bundled suite and hands that responsibility to you. Our pick was [Mocha](https://mochajs.org/), a fairly generic runner with no ties to WebDrivers, which means the script has to put in some effort wiring everything up in the right sequence. Since [Mocha](https://mochajs.org/) looks for its test file at `test/test.js` out of the box, we'll go ahead and add that file.

`test/test.js`:

```
import os from 'os';import path from 'path';import { expect } from 'chai';import { spawn, spawnSync } from 'child_process';import { Builder, By, Capabilities } from 'selenium-webdriver';import { fileURLToPath } from 'url';
const __dirname = fileURLToPath(new URL('.', import.meta.url));
// create the path to the expected application binaryconst application = path.resolve(  __dirname,  '..',  '..',  'src-tauri',  'target',  'debug',  'tauri-app');
// keep track of the webdriver instance we createlet driver;
// keep track of the tauri-driver process we startlet tauriDriver;let exit = false;
before(async function () {  // set timeout to 2 minutes to allow the program to build if it needs to  this.timeout(120000);
  // ensure the app has been built  spawnSync('yarn', ['tauri', 'build', '--debug', '--no-bundle'], {    cwd: path.resolve(__dirname, '../..'),    stdio: 'inherit',    shell: true,  });
  // start tauri-driver  tauriDriver = spawn(    path.resolve(os.homedir(), '.cargo', 'bin', 'tauri-driver'),    [],    { stdio: [null, process.stdout, process.stderr] }  );  tauriDriver.on('error', (error) => {    console.error('tauri-driver error:', error);    process.exit(1);  });  tauriDriver.on('exit', (code) => {    if (!exit) {      console.error('tauri-driver exited with code:', code);      process.exit(1);    }  });
  const capabilities = new Capabilities();  capabilities.set('tauri:options', { application });  capabilities.setBrowserName('wry');
  // start the webdriver client  driver = await new Builder()    .withCapabilities(capabilities)    .usingServer('http://127.0.0.1:4444/')    .build();});
after(async function () {  // stop the webdriver session  await closeTauriDriver();});
describe('Hello Tauri', () => {  it('should be cordial', async () => {    const text = await driver.findElement(By.css('body > h1')).getText();    expect(text).to.match(/^[hH]ello/);  });
  it('should be excited', async () => {    const text = await driver.findElement(By.css('body > h1')).getText();    expect(text).to.match(/!$/);  });
  it('should be easy on the eyes', async () => {    // selenium returns color css values as rgb(r, g, b)    const text = await driver      .findElement(By.css('body'))      .getCssValue('background-color');
    const rgb = text.match(/^rgb\((?<r>\d+), (?<g>\d+), (?<b>\d+)\)$/).groups;    expect(rgb).to.have.all.keys('r', 'g', 'b');
    const luma = 0.2126 * rgb.r + 0.7152 * rgb.g + 0.0722 * rgb.b;    expect(luma).to.be.lessThan(100);  });});
async function closeTauriDriver() {  exit = true;  // kill the tauri-driver process  tauriDriver.kill();  // stop the webdriver session  await driver.quit();}
function onShutdown(fn) {  const cleanup = () => {    try {      fn();    } finally {      process.exit();    }  };
  process.on('exit', cleanup);  process.on('SIGINT', cleanup);  process.on('SIGTERM', cleanup);  process.on('SIGHUP', cleanup);  process.on('SIGBREAK', cleanup);}
onShutdown(() => {  closeTauriDriver();});
```

Anyone who has used JS testing frameworks will recognize `describe`, `it`, and `expect` right away. There are also moderately involved `before()` and `after()` hooks handling the startup and cleanup of mocha. Every line outside the actual tests carries a comment clarifying what the setup and teardown code does. Compared with the Spec file from the [WebdriverIO example](https://v2.tauri.app/develop/tests/webdriver/example/webdriverio/#spec), you'll spot considerably more non-test code here, because a handful of extra WebDriver pieces have to be wired up.

### Running the Test Suite

With the dependencies in place and the test script ready, it's time to fire it off!

- npm
- yarn

```
npm test
```

```
yarn test
```

What lands on screen should resemble the following:

```
➜  selenium git:(main) ✗ yarn testyarn run v1.22.11$ Mocha

  Hello Tauri    ✔ should be cordial (120ms)    ✔ should be excited    ✔ should be easy on the eyes

  3 passing (588ms)
Done in 0.93s.
```

As shown, the `Hello Tauri` suite we declared via `describe` saw all 3 cases we wrote with `it` come back green!

By pairing [Selenium](https://selenium.dev/) with a bit of test-suite plumbing, we've stood up e2e testing while leaving our Tauri application entirely untouched!

---

## WebdriverIO

### WebdriverIO

This WebDriver walkthrough relies on [WebdriverIO](https://webdriver.io/) and the suite it brings along. We presume Node.js is already set up, together with `npm` or `yarn`, though the [finished example project](https://github.com/tauri-apps/webdriver-example) leans on `pnpm` instead.

### Create a Directory for the Tests

Let's set aside a place inside the project where these tests can live. A nested folder is used in this sample because more frameworks are covered later, but ordinarily just one suffices. Make the folder via `mkdir e2e-tests`. Everything that follows presumes your working directory is `e2e-tests`.

### Initializing a WebdriverIO Project

A ready-made `package.json` serves to bootstrap this suite, given that the [WebdriverIO](https://webdriver.io/) configuration choices have already been made and we want to demonstrate an uncomplicated, working setup. A folded-up walkthrough on building it from the ground up sits at the foot of this section.

`package.json`:

```
{  "name": "webdriverio",  "version": "1.0.0",  "private": true,  "type": "module",  "scripts": {    "test": "wdio run wdio.conf.js"  },  "dependencies": {    "@wdio/cli": "^9.19.0"  },  "devDependencies": {    "@wdio/local-runner": "^9.19.0,    "@wdio/mocha-framework": "^9.19.0",    "@wdio/spec-reporter": "^9.19.0"  }}
```

There's a script that executes a [WebdriverIO](https://webdriver.io/) config as the suite, surfaced behind the `test` command. The assorted dependencies were pulled in by the `@wdio/cli` command back when we initialized things. Put briefly, they cover the most basic arrangement: a local WebDriver runner, [Mocha](https://mochajs.org/) serving as the framework, and a plain Spec Reporter.

Open me to walk through configuring a project from the ground up

Because the CLI runs interactively, the tooling is yours to pick. Bear in mind this will probably take you off the path the guide follows, leaving you to reconcile those discrepancies on your own.

Now let's bring the [WebdriverIO](https://webdriver.io/) CLI into this npm project.

- npm
- yarn

```
npm install @wdio/cli
```

```
yarn add @wdio/cli
```

Afterward, to launch the interactive config step that scaffolds a [WebdriverIO](https://webdriver.io/) suite, issue:

- npm
- yarn

```
npx wdio config
```

```
yarn wdio config
```

### Config

As you might have spotted, the `test` script inside `package.json` points at a file named `wdio.conf.js`. This is the [WebdriverIO](https://webdriver.io/) config that governs nearly every facet of the suite.

`wdio.conf.js`:

```
import os from 'os';import path from 'path';import { spawn, spawnSync } from 'child_process';import { fileURLToPath } from 'url';
const __dirname = fileURLToPath(new URL('.', import.meta.url));
// keep track of the `tauri-driver` child processlet tauriDriver;let exit = false;
export const config = {  host: '127.0.0.1',  port: 4444,  specs: ['./develop/tests/specs/**/*.js'],  maxInstances: 1,  capabilities: [    {      maxInstances: 1,      'tauri:options': {        application: '../src-tauri/target/debug/tauri-app',      },    },  ],  reporters: ['spec'],  framework: 'mocha',  mochaOpts: {    ui: 'bdd',    timeout: 60000,  },
  // ensure the rust project is built since we expect this binary to exist for the webdriver sessions  onPrepare: () => {    // Remove the extra `--` if you're not using npm!    spawnSync(      'npm',      ['run', 'tauri', 'build', '--', '--debug', '--no-bundle'],      {        cwd: path.resolve(__dirname, '..'),        stdio: 'inherit',        shell: true,      }    );  },
  // ensure we are running `tauri-driver` before the session starts so that we can proxy the webdriver requests  beforeSession: () => {    tauriDriver = spawn(      path.resolve(os.homedir(), '.cargo', 'bin', 'tauri-driver'),      [],      { stdio: [null, process.stdout, process.stderr] }    );
    tauriDriver.on('error', (error) => {      console.error('tauri-driver error:', error);      process.exit(1);    });    tauriDriver.on('exit', (code) => {      if (!exit) {        console.error('tauri-driver exited with code:', code);        process.exit(1);      }    });  },
  // clean up the `tauri-driver` process we spawned at the start of the session  // note that afterSession might not run if the session fails to start, so we also run the cleanup on shutdown  afterSession: () => {    closeTauriDriver();  },};
function closeTauriDriver() {  exit = true;  tauriDriver?.kill();}
function onShutdown(fn) {  const cleanup = () => {    try {      fn();    } finally {      process.exit();    }  };
  process.on('exit', cleanup);  process.on('SIGINT', cleanup);  process.on('SIGTERM', cleanup);  process.on('SIGHUP', cleanup);  process.on('SIGBREAK', cleanup);}
// ensure tauri-driver is closed when our test process exitsonShutdown(() => {  closeTauriDriver();});
```

Should the fields on the `config` object pique your curiosity, we [suggest reading the documentation](https://webdriver.io/docs/configurationfile). For the parts that aren't WDIO-specific, comments lay out the reasoning behind the commands inside `onPrepare`, `beforeSession`, and `afterSession`. Our specs point at `"./test/specs/**/*.js"`, so the next move is to author a spec.

### Spec

The code that genuinely exercises your application lives inside a spec. These specs get picked up by the test runner, which executes them however it deems appropriate. Let's write our spec inside the folder we pointed to earlier.

`test/specs/example.e2e.js`:

```
// calculates the luma from a hex color `#abcdef`function luma(hex) {  if (hex.startsWith('#')) {    hex = hex.substring(1);  }
  const rgb = parseInt(hex, 16);  const r = (rgb >> 16) & 0xff;  const g = (rgb >> 8) & 0xff;  const b = (rgb >> 0) & 0xff;  return 0.2126 * r + 0.7152 * g + 0.0722 * b;}
describe('Hello Tauri', () => {  it('should be cordial', async () => {    const header = await $('body > h1');    const text = await header.getText();    expect(text).toMatch(/^[hH]ello/);  });
  it('should be excited', async () => {    const header = await $('body > h1');    const text = await header.getText();    expect(text).toMatch(/!$/);  });
  it('should be easy on the eyes', async () => {    const body = await $('body');    const backgroundColor = await body.getCSSProperty('background-color');    expect(luma(backgroundColor.parsed.hex)).toBeLessThan(100);  });});
```

That `luma` function up top merely assists one of the tests and has nothing to do with exercising the application itself. Coming from other testing frameworks, you'll likely recognize comparable exposed helpers in use here, namely `describe`, `it`, and `expect`. As for the remaining APIs, things such as `$` and the methods it offers, those are documented in the [WebdriverIO API docs](https://webdriver.io/docs/api).

### Running the Test Suite

Having both the config and a spec squared away, let's go ahead and run it!

- npm
- yarn

```
npm test
```

```
yarn test
```

The output we get should look something like this:

```
➜  webdriverio git:(main) ✗ yarn testyarn run v1.22.11$ wdio run wdio.conf.js
Execution of 1 workers started at 2021-08-17T08:06:10.279Z
[0-0] RUNNING in undefined - /develop/tests/specs/example.e2e.js[0-0] PASSED in undefined - /develop/tests/specs/example.e2e.js
 "spec" Reporter:------------------------------------------------------------------[wry 0.12.1 linux #0-0] Running: wry (v0.12.1) on linux[wry 0.12.1 linux #0-0] Session ID: 81e0107b-4d38-4eed-9b10-ee80ca47bb83[wry 0.12.1 linux #0-0][wry 0.12.1 linux #0-0] » /develop/tests/specs/example.e2e.js[wry 0.12.1 linux #0-0] Hello Tauri[wry 0.12.1 linux #0-0]    ✓ should be cordial[wry 0.12.1 linux #0-0]    ✓ should be excited[wry 0.12.1 linux #0-0]    ✓ should be easy on the eyes[wry 0.12.1 linux #0-0][wry 0.12.1 linux #0-0] 3 passing (244ms)

Spec Files:   1 passed, 1 total (100% completed) in 00:00:01
Done in 1.98s.
```

The Spec Reporter reports back that every one of the 3 tests in the `test/specs/example.e2e.js` file passed, capped off by the summary line `Spec Files: 1 passed, 1 total (100% completed) in 00:00:01`.

Thanks to the [WebdriverIO](https://webdriver.io/) suite, we've effortlessly brought e2e testing to our Tauri application off the back of a handful of config lines plus one command to launch it! Better still, the application itself needed no changes whatsoever.

---
