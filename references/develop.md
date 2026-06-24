# Configuration Files

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Develop; Configuration Files.

## Develop

### Develop

With [everything set up](https://v2.tauri.app/start/) in place, the next step is launching your project through Tauri.

When your stack includes a UI framework or a JavaScript bundler, a dev server is usually available to accelerate iteration. If you have not yet pointed Tauri at your app's dev URL and the script that boots it, supply those through the [devUrl](https://v2.tauri.app/reference/config/#devurl) and [beforeDevCommand](https://v2.tauri.app/reference/config/#beforedevcommand) config values:

tauri.conf.json

```
{  "build": {    "devUrl": "http://localhost:3000",    "beforeDevCommand": "npm run dev"  }}
```

If instead you work without any UI framework or module bundler, simply direct Tauri to where your frontend source lives, and the Tauri CLI will spin up a development server on your behalf:

tauri.conf.json

```
{  "build": {    "frontendDist": "./src"  }}
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

```
npm run tauri dev
```

```
yarn tauri dev
```

```
pnpm tauri dev
```

```
deno task tauri dev
```

```
bun tauri dev
```

```
cargo tauri dev
```

On your initial run, expect Rust's package manager to spend **several minutes** fetching and compiling every dependency it needs. Those artifacts get cached afterward, so later builds finish far quicker since only your own changes are recompiled.

After Rust wraps up the build, a webview pops up showing your web app. Edit the web app as you like, and provided your tooling permits it, the webview will refresh on its own, much the way a browser does.

#### Opening the Web Inspector

To debug, bring up the Web Inspector either by right-clicking inside the webview and selecting “Inspect,” or through the `Ctrl + Shift + I` shortcut on Windows and Linux, or the `Cmd + Option + I` shortcut on macOS.

### Developing Your Mobile Application

The mobile workflow mirrors the desktop one closely, except the command you launch is `tauri android dev` or `tauri ios dev`:

- npm
- yarn
- pnpm
- deno
- bun
- cargo

```
npm run tauri [android|ios] dev
```

```
yarn tauri [android|ios] dev
```

```
pnpm tauri [android|ios] dev
```

```
deno task tauri [android|ios] dev
```

```
bun tauri [android|ios] dev
```

```
cargo tauri [android|ios] dev
```

During the very first invocation, Cargo's package manager could take **several minutes** to retrieve and compile all the crates this requires. Because those results are kept in a cache, every run after that is noticeably quicker, recompiling only what you changed.

#### Development Server

On mobile the dev server behaves much like its desktop counterpart, but targeting a physical iOS device means binding it to a specific address that the Tauri CLI supplies through the `TAURI_DEV_HOST` environment variable. That address is by default a public network address, or alternatively the device's own TUN address — a more secure choice that for now relies on Xcode to reach the device.

Going the TUN-address route requires launching Xcode ahead of the dev command and confirming the device shows up over the network under the Window > Devices and Simulators menu. From there, run `tauri ios dev --force-ip-prompt` and pick the iOS device address (an IPv6 address ending in **::2**).

So that the dev server binds to the right host and the iOS device can reach it, adjust its config to honor the `TAURI_DEV_HOST` value whenever that value is set. Below is a sample Vite configuration:

```
import { defineConfig } from 'vite';
const host = process.env.TAURI_DEV_HOST;
// https://vitejs.dev/config/export default defineConfig({  clearScreen: false,  server: {    host: host || false,    port: 1420,    strictPort: true,    hmr: host      ? {          protocol: 'ws',          host,          port: 1421,        }      : undefined,  },});
```

Consult the setup guide for your particular framework if you need further detail.

#### Device Selection

Out of the box, the mobile dev command attempts to launch your app on whatever device is plugged in, and if none is found it asks you to pick a simulator. Should you wish to set the run target ahead of time, pass the device or simulator name as an argument:

- npm
- yarn
- pnpm
- deno
- bun
- cargo

```
npm run tauri ios dev 'iPhone 15'
```

```
yarn tauri ios dev 'iPhone 15'
```

```
pnpm tauri ios dev 'iPhone 15'
```

```
deno task tauri ios dev 'iPhone 15'
```

```
bun tauri ios dev 'iPhone 15'
```

```
cargo tauri ios dev 'iPhone 15'
```

#### Using Xcode or Android Studio

Another option is to drive development through Xcode or Android Studio directly. Leaning on the IDE rather than the command line can make certain development problems easier to diagnose. When you want the mobile IDE to open instead of launching on an attached device or simulator, add the `--open` flag:

- npm
- yarn
- pnpm
- deno
- bun
- cargo

```
npm run tauri [android|ios] dev --open
```

```
yarn tauri [android|ios] dev --open
```

```
pnpm tauri [android|ios] dev --open
```

```
deno task tauri [android|ios] dev --open
```

```
bun tauri [android|ios] dev --open
```

```
cargo tauri [android|ios] dev --open
```

#### Opening the Web Inspector

-

iOS

Reaching the Web Inspector for an iOS app is only possible through Safari.

Launch Safari on your Mac, pick **Safari > Settings** from the menu bar, open **Advanced**, and tick **Show features for web developers**.

When the target is a physical device, you also have to switch on **Web Inspector** under **Settings > Safari > Advanced**.

Once every step is done, a **Develop** menu appears in Safari listing the attached devices and apps available for inspection. Choose your device or simulator, then click **localhost** to bring up the Safari Developer Tools window.

-

Android

Android emulators come with the inspector turned on already, yet physical hardware needs it switched on manually. Plug your Android device into the computer, launch the **Settings** app there, tap **About**, find Build Number, and tap it 7 times. Doing so unlocks Developer Mode along with the **Developer Options** settings on the device.

Next, to permit app debugging, head into the **Developer Options** settings, flip the developer options switch on, and turn on **USB Debugging**.

Android's Web Inspector runs on Google Chrome's DevTools, which you reach by visiting `chrome://inspect` in Chrome on your machine. While your Android app is live, the device or emulator shows up in the remote devices list, and clicking **inspect** beside the matching entry launches the developer tools.

#### Troubleshooting

1. Error running build script on Xcode

Tauri ties into the iOS Xcode project by adding a build phase that runs the Tauri CLI to compile the Rust source into a library loaded at runtime. Since that build phase runs within the Xcode process context, it may not pick up shell tweaks like PATH additions, so take care with utilities such as Node.js version managers that might not work in that environment.

1. Network permission prompt on first iOS app execution

The first run of `tauri ios dev` may trigger an iOS prompt asking permission to discover and connect to devices on your local network. That prompt appears because reaching your dev server from an iOS device requires the server to be reachable across the local network. Click Allow and relaunch your application to get it running on the device.

### Reacting to Source Code Changes

Just as your webview mirrors edits as they happen, `tauri dev` keeps an eye on the `src-tauri` folder along with its dependent crates across the workspace, so any change there triggers an automatic rebuild and restart of your application.

If you would rather turn this off, attach the `--no-watch` flag to the `tauri dev` command.

When you want particular files left out of the watch, set up `.taurignore` files, which behave just like ordinary `.gitignore` files:

```
build/src/generated/*.rsdeny.toml
```

Typically these `.taurignore` files live in the `src-tauri` directory or at the [cargo workspace](https://doc.rust-lang.org/cargo/reference/workspaces.html) root. As things stand, `tauri dev` searches for `.taurignore` files throughout the shared ancestor of the watched folders and the Cargo workspace root.

### Using the Browser DevTools

Because Tauri's APIs function solely inside your app window, the moment you begin relying on them you can no longer load your frontend in your operating system's browser.

Should you favor your browser's own developer tooling, set up [tauri-invoke-http](https://github.com/tauri-apps/tauri-invoke-http) to route Tauri API calls over an HTTP server.

### Source Control

Within your project repository, you **SHOULD** check `src-tauri/Cargo.lock` into git alongside `src-tauri/Cargo.toml`, since Cargo relies on the lockfile to make builds deterministic. For that reason, every application is advised to track its `Cargo.lock`. By contrast, you **SHOULD NOT** commit the `src-tauri/target` folder or anything it holds.

---

## Configuration Files

### Configuration Files

Because Tauri serves as a toolkit for assembling applications, a project's settings can be spread across several files. The ones you will most often encounter are `tauri.conf.json`, `package.json` and `Cargo.toml`. This page gives a short rundown of each so you know which file to reach for.

### Tauri Config

Through the Tauri configuration you specify where your Web app comes from, supply your application's metadata, set up bundles, declare plugin settings, and shape runtime behavior by defining windows, tray icons, menus and so on.

Both the Tauri runtime and the Tauri CLI consume this file. It lets you lay out build settings (for instance the [command run before `tauri build`](https://v2.tauri.app/reference/config/#beforebuildcommand) or [`tauri dev`](https://v2.tauri.app/reference/config/#beforedevcommand-1) kicks in), assign the [name](https://v2.tauri.app/reference/config/#productname) and [version of your app](https://v2.tauri.app/reference/config/#version), [control the Tauri runtime](https://v2.tauri.app/reference/config/#appconfig), and [configure plugins](https://v2.tauri.app/reference/config/#plugins).

### Supported Formats

JSON is what Tauri uses for its config by default. To switch to JSON5 or TOML, add the matching `config-json5` or `config-toml` feature flag onto the `tauri` and `tauri-build` dependencies inside `Cargo.toml`.

Cargo.toml

```
[build-dependencies]tauri-build = { version = "2.0.0", features = [ "config-json5" ] }
[dependencies]tauri = { version = "2.0.0", features = [  "config-json5" ] }
```

While the layout and the values stay identical no matter the format, the syntax you write must follow the conventions of each particular file's format:

tauri.conf.json

```
{  build: {    devUrl: 'http://localhost:3000',    // start the dev server    beforeDevCommand: 'npm run dev',  },  bundle: {    active: true,    icon: ['icons/app.png'],  },  app: {    windows: [      {        title: 'MyApp',      },    ],  },  plugins: {    updater: {      pubkey: 'updater pub key',      endpoints: ['https://my.app.updater/{{target}}/{{current_version}}'],    },  },}
```

Tauri.toml

```
[build]dev-url = "http://localhost:3000"# start the dev serverbefore-dev-command = "npm run dev"
[bundle]active = trueicon = ["icons/app.png"]
[[app.windows]]title = "MyApp"
[plugins.updater]pubkey = "updater pub key"endpoints = ["https://my.app.updater/{{target}}/{{current_version}}"]
```

Bear in mind that comments are allowed in JSON5 and TOML, and TOML lets you write config names in the more idiomatic kebab-case. Across all 3 formats, field names remain case-sensitive.

### Platform-specific Configuration

Beyond the default configuration file, Tauri is able to load a platform-specific configuration drawn from:

- `tauri.linux.conf.json` or `Tauri.linux.toml` for Linux
- `tauri.windows.conf.json` or `Tauri.windows.toml` for Windows
- `tauri.macos.conf.json` or `Tauri.macos.toml` for macOS
- `tauri.android.conf.json` or `Tauri.android.toml` for Android
- `tauri.ios.conf.json` or `Tauri.ios.toml` for iOS

Following the [JSON Merge Patch (RFC 7396)](https://datatracker.ietf.org/doc/html/rfc7396) specification, the platform-specific configuration file is merged into the main configuration object.

As an illustration, suppose you start from this base `tauri.conf.json`:

tauri.conf.json

```
{  "productName": "MyApp",  "bundle": {    "resources": ["./resources"]  },  "plugins": {    "deep-link": {}  }}
```

Together with this `tauri.linux.conf.json`:

tauri.linux.conf.json

```
{  "productName": "my-app",  "bundle": {    "resources": ["./linux-assets"]  },  "plugins": {    "cli": {      "description": "My app",      "subcommands": {        "update": {}      }    },    "deep-link": {}  }}
```

The configuration that Linux ends up resolving to is this object:

```
{  "productName": "my-app",  "bundle": {    "resources": ["./linux-assets"]  },  "plugins": {    "cli": {      "description": "My app",      "subcommands": {        "update": {}      }    },    "deep-link": {}  }}
```

On top of that, the CLI lets you hand in a configuration to be merged; the next section covers the details.

### Extending the Configuration

When you invoke any of the `dev`, `android dev`, `ios dev`, `build`, `android build`, `ios build` or `bundle` commands, the Tauri CLI lets you augment the Tauri configuration. You pass this extension through the `--config` argument, supplying either a raw JSON string or a path pointing to a JSON file. To combine the value you give with the configuration object that was originally resolved, Tauri applies the [JSON Merge Patch (RFC 7396)](https://datatracker.ietf.org/doc/html/rfc7396) specification.

You can lean on this approach to spin up several variants of your application or to gain extra leeway in how you configure your application bundles.

To take one case, shipping a fully separate _beta_ application is possible by using this feature to set a distinct application name and identifier:

src-tauri/tauri.beta.conf.json

```
{  "productName": "My App Beta",  "identifier": "com.myorg.myappbeta"}
```

Then, to ship that standalone _beta_ app, hand over this configuration file at build time:

- npm
- yarn
- pnpm
- deno
- bun
- cargo

```
npm run tauri build -- --config src-tauri/tauri.beta.conf.json
```

```
yarn tauri build --config src-tauri/tauri.beta.conf.json
```

```
pnpm tauri build --config src-tauri/tauri.beta.conf.json
```

```
deno task tauri build --config src-tauri/tauri.beta.conf.json
```

```
bun tauri build --config src-tauri/tauri.beta.conf.json
```

```
cargo tauri build --config src-tauri/tauri.beta.conf.json
```

### `Cargo.toml`

Within Cargo's manifest file you list the Rust crates your app relies on, your app's metadata, and assorted Rust-specific features. Unless you plan to write your app's backend in Rust, you probably won't touch it often, yet it's worth understanding that it's there and what role it plays.

Here is a stripped-down sample `Cargo.toml` file for a Tauri project:

Cargo.toml

```
[package]name = "app"version = "0.1.0"description = "A Tauri App"authors = ["you"]license = ""repository = ""default-run = "app"edition = "2021"rust-version = "1.57"
[build-dependencies]tauri-build = { version = "2.0.0" }
[dependencies]serde_json = "1.0"serde = { version = "1.0", features = ["derive"] }tauri = { version = "2.0.0", features = [ ] }
```

The pieces that deserve the most attention are the `tauri-build` and `tauri` dependencies. As a rule, both ought to sit on the same latest minor versions as the Tauri CLI, although that's not an absolute requirement. Whenever your app misbehaves at runtime, verify that each Tauri version (`tauri` and `tauri-cli`) is current for its respective minor release.

Cargo's version numbers follow [Semantic Versioning](https://semver.org). Invoking `cargo update` from within the `src-tauri` folder fetches the newest Semver-compatible version of every dependency. As an example, declaring `2.0.0` as the version of `tauri-build` leads Cargo to find and grab version `2.0.1`, that being the most recent Semver-compatible release on offer. Tauri only bumps the major version number on a breaking change, so you can count on safely moving up to the latest minor and patch versions without worrying that your code will break.

Should you need to pin a particular crate version, you can require exact versions by putting `=` in front of the dependency's version number:

```
tauri-build = { version = "=2.0.0" }
```

One more thing worth noticing is the `features=[]` segment of the `tauri` dependency. Both `tauri dev` and `tauri build` figure out on their own which features your project requires, drawing on your Tauri configuration. The [documentation](https://docs.rs/tauri/2.0.0/tauri/#cargo-features) goes deeper on `tauri` feature flags.

Building your application generates a `Cargo.lock` file. Its main job is to guarantee that identical dependencies are used on every machine throughout development (much like `yarn.lock`, `pnpm-lock.yaml` or `package-lock.json` over in Node.js). Committing this file to your source repository is advised so that your builds stay consistent.

For a fuller account of the Cargo manifest file, consult the [official documentation](https://doc.rust-lang.org/cargo/reference/manifest.html).

### `package.json`

This serves as Node.js's package file. When your Tauri app's frontend is built on Node.js-based technologies (think `npm`, `yarn`, or `pnpm`), this file is where you set up the frontend dependencies and scripts.

A minimal `package.json` file for a Tauri project could resemble something along these lines:

package.json

```
{  "scripts": {    "dev": "command to start your app development mode",    "build": "command to build your app frontend",    "tauri": "tauri"  },  "dependencies": {    "@tauri-apps/api": "^2.0.0",    "@tauri-apps/cli": "^2.0.0"  }}
```

People often rely on the `"scripts"` section to hold the commands that start and build the frontend powering their Tauri application. In the `package.json` shown above, the `dev` command (run via `yarn dev` or `npm run dev`) boots the frontend framework, while the `build` command (run via `yarn build` or `npm run build`) compiles your frontend's Web assets for Tauri to bundle in production. The handiest way to wire these scripts up is to connect them to the Tauri CLI through the Tauri configuration's [beforeDevCommand](https://v2.tauri.app/reference/config/#beforedevcommand-1) and [beforeBuildCommand](https://v2.tauri.app/reference/config/#beforebuildcommand) hooks:

tauri.conf.json

```
{  "build": {    "beforeDevCommand": "yarn dev",    "beforeBuildCommand": "yarn build"  }}
```

The dependencies object lays out which packages Node.js needs to fetch when you run `yarn`, `pnpm install` or `npm install` — here that means the Tauri CLI and API.

Alongside the `package.json` file you might also find a `yarn.lock`, `pnpm-lock.yaml` or `package-lock.json` file. Their purpose is to make sure that any later install of the dependencies hands you precisely the versions you relied on during development (akin to `Cargo.lock` in Rust).

For more on the `package.json` file format, see the [official documentation](https://docs.npmjs.com/cli/v8/configuring-npm/package-json).

---
