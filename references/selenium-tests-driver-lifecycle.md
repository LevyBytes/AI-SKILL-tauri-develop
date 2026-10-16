# Selenium assertions and test-suite lifecycle

[Topic context](webdriver-testing-index.md) / [Previous source section](selenium-test-project-setup.md) / [Next source section](webdriverio-test-project-setup.md)

Context: Tauri v2: Selenium assertions and test-suite lifecycle. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-testing"></a> <a id="source-running-the-test-suite"></a>

### Testing

In contrast to the [WebdriverIO Test Suite](https://v2.tauri.app/develop/tests/webdriver/example/webdriverio/#config), Selenium ships without any bundled suite and hands that responsibility to you. Our pick was [Mocha](https://mochajs.org/), a fairly generic runner with no ties to WebDrivers, which means the script has to put in some effort wiring everything up in the right sequence. Since [Mocha](https://mochajs.org/) looks for its test file at `test/test.js` out of the box, we'll go ahead and add that file.


Create both modules under `e2e-tests/test/`; neither example file is a continuation fragment.

- [Driver lifecycle](selenium-driver-lifecycle.md): Open when building the app, starting the driver or handling cleanup.
- [Application assertions](selenium-app-assertions.md): Open when defining the Mocha tests that use that driver.
- [Run the suite](selenium-test-execution.md): Open when executing the tests and interpreting their output.
