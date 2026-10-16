# WebDriver platform requirements and driver setup

[Topic context](webdriver-testing-index.md) / [Next source section](webdriver-github-actions.md)

Context: Tauri v2: WebDriver platform requirements and driver setup. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-ci-example"></a> <a id="source-overview"></a> <a id="source-webdriver"></a> <a id="source-webdriver-1"></a> <a id="source-system-dependencies"></a> <a id="source-linux"></a> <a id="source-windows"></a> <a id="source-example-applications"></a> <a id="source-continuous-integration-ci"></a>

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

```shell
cargo install tauri-driver --locked
```

Since the tooling relies on each system's built-in [WebDriver](https://www.w3.org/TR/webdriver/) server, every supported platform imposes its own prerequisites before [`tauri-driver`](https://crates.io/crates/tauri-driver) will work.

### Linux

On Linux the driver of choice is `WebKitWebDriver`. To find out whether it is already present, run `which WebKitWebDriver`; certain distributions ship it inside their main WebKit package. Elsewhere it lives in its own package instead, for instance `webkit2gtk-driver` on Debian-derived systems.

### Windows

Be sure the [Microsoft Edge Driver](https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/) you obtain lines up with the Edge release present on the Windows machine where your app gets built and exercised. For a current Windows setup that is nearly always the newest stable build. A mismatch between the pair tends to leave your WebDriver suite stuck as it attempts to establish a connection.

Fetching the right Microsoft Edge Driver can be handled for you by the [msedgedriver-tool](https://github.com/chippers/msedgedriver-tool):

Terminal window

```powershell
cargo install --git https://github.com/chippers/msedgedriver-tool
& "$HOME/.cargo/bin/msedgedriver-tool.exe"
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

