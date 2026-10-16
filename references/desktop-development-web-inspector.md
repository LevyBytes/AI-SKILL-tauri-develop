# Desktop development and Web Inspector

[Topic context](development-configuration-index.md) / [Next source section](mobile-server-device-selection.md)

Context: Tauri v2: Desktop development and Web Inspector. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-configuration-files"></a> <a id="source-overview"></a> <a id="source-develop"></a> <a id="source-develop-1"></a> <a id="source-developing-your-desktop-application"></a> <a id="source-opening-the-web-inspector"></a>

# Configuration Files

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Develop; Configuration Files.

## Develop

### Develop

With [everything set up](https://v2.tauri.app/start/) in place, the next step is launching your project through Tauri.

When your stack includes a UI framework or a JavaScript bundler, a dev server is usually available to accelerate iteration. If you have not yet pointed Tauri at your app's dev URL and the script that boots it, supply those through the [devUrl](https://v2.tauri.app/reference/config/#devurl) and [beforeDevCommand](https://v2.tauri.app/reference/config/#beforedevcommand) config values:

tauri.conf.json

```json
{
  "build": {
    "devUrl": "http://localhost:3000",
    "beforeDevCommand": "npm run dev"
  }
}
```

If instead you work without any UI framework or module bundler, simply direct Tauri to where your frontend source lives, and the Tauri CLI will spin up a development server on your behalf:

tauri.conf.json

```json
{
  "build": {
    "frontendDist": "./src"
  }
}
```

Keep in mind that for this case the `src` folder needs to contain an `index.html` file plus whatever additional assets your frontend pulls in.

### Developing Your Desktop Application

For desktop work, invoke the `tauri dev` command.

- npm
- yarn
- pnpm
- deno
- bun
- cargo

```sh
npm run tauri dev
```

```sh
yarn tauri dev
```

```sh
pnpm tauri dev
```

```sh
deno task tauri dev
```

```sh
bun tauri dev
```

```sh
cargo tauri dev
```

On your initial run, expect Rust's package manager to spend **several minutes** fetching and compiling every dependency it needs. Those artifacts get cached afterward, so later builds finish far quicker since only your own changes are recompiled.

After Rust wraps up the build, a webview pops up showing your web app. Edit the web app as you like, and provided your tooling permits it, the webview will refresh on its own, much the way a browser does.

#### Opening the Web Inspector

To debug, bring up the Web Inspector either by right-clicking inside the webview and selecting “Inspect,” or through the `Ctrl + Shift + I` shortcut on Windows and Linux, or the `Cmd + Option + I` shortcut on macOS.

