# WebdriverIO configuration, specs and execution

[Topic context](webdriver-testing-index.md) / [Previous source section](webdriverio-test-project-setup.md)

Context: Tauri v2: WebdriverIO configuration, specs and execution. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-config"></a> <a id="source-spec"></a> <a id="source-running-the-test-suite-1"></a>

### Config

As you might have spotted, the `test` script inside `package.json` points at a file named `wdio.conf.js`. This is the [WebdriverIO](https://webdriver.io/) config that governs nearly every facet of the suite.

`wdio.conf.js`:


- [Runner configuration](webdriverio-runner-configuration.md): Open when wiring the build and driver lifecycle into WDIO.
- [Application spec](webdriverio-application-spec.md): Open when writing the test cases selected by the configuration.
- [Run the suite](webdriverio-test-execution.md): Open when executing WDIO and interpreting its report.
