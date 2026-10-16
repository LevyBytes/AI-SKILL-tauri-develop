# WebDriver continuous integration

[Topic context](webdriver-testing-index.md) / [Previous source section](webdriver-platform-driver-setup.md) / [Next source section](selenium-test-project-setup.md)

Context: Tauri v2: WebDriver continuous integration. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-continuous-integration"></a> <a id="source-continuous-integration-1"></a> <a id="source-github-actions"></a>

## Continuous Integration

### Continuous Integration

Your CI pipeline can drive [WebDriver](https://www.w3.org/TR/webdriver/) tests through [`tauri-driver`](https://crates.io/crates/tauri-driver). The walkthrough here builds on the [WebdriverIO](https://webdriver.io/) sample we [previously built together](https://v2.tauri.app/develop/tests/webdriver/example/webdriverio/), paired with GitHub Actions.

On Linux these WebDriver runs rely on a simulated display. Certain providers, GitHub Actions among them, will additionally let you exercise WebDriver tests on Windows.

### GitHub Actions

This GitHub Actions setup takes the following for granted:

1. Your Tauri app lives under the `src-tauri` folder.
2. The [WebDriverIO](https://webdriver.io/) runner sits in the `e2e-tests` directory and fires off whenever `yarn test` is invoked from there.
   .github/workflows/webdriver.yml

[Complete workflow](webdriver-github-actions-workflow.md): Open when configuring the job, dependencies and test steps.
