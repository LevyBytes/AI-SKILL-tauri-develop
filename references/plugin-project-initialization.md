# Initialize a plugin project

Open when naming and scaffolding a plugin or finding its generated files.

[Topic context](plugin-initialization-naming-configuration.md)

# Plugins

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

### Plugin Development

Through a plugin you can tap into the Tauri lifecycle, surface Rust functionality that leans on the web view APIs, service commands written in Rust, Kotlin, or Swift, and a great deal beyond that.

What Tauri provides is a windowing layer with web view support, a channel for passing messages back and forth between the Rust process and the web view, and an event system paired with a handful of utilities meant to make building easier. The Tauri core is deliberately kept lean, leaving out capabilities that aren't universally required. In their place it supplies a way to bolt extra functionality onto a Tauri application, and that mechanism is the plugin.

Each Tauri plugin pairs a Cargo crate with an optional NPM package supplying API bindings for the plugin's commands and events. A plugin project may also bundle an Android library project together with a Swift package targeting iOS. For details on building plugins that target Android and iOS, see the [Mobile Plugin Development guide](https://v2.tauri.app/develop/plugins/develop-mobile/).

### Naming Convention

A Tauri plugin's name carries a prefix in front of it. You declare that plugin name within the plugin configuration at [`tauri.conf.json > plugins`](https://v2.tauri.app/reference/config/#pluginconfig).

Out of the box, Tauri puts the `tauri-plugin-` prefix on your plugin crate. Doing so makes the plugin easier for the Tauri community to find and lets the Tauri CLI work with it. As you set up a fresh plugin project, you have to supply its name. The crate that gets generated is named `tauri-plugin-{plugin-name}`, while the JavaScript NPM package becomes `tauri-plugin-{plugin-name}-api` (though where you can, we suggest reaching for an [NPM scope](https://docs.npmjs.com/about-scopes)). For NPM packages, Tauri's convention names them `@scope-name/plugin-{plugin-name}`.

### Initialize Plugin Project

Scaffold a fresh plugin project by running `plugin new`. Should you have no need for the NPM package, pass the `--no-api` CLI flag. To set the plugin up with Android support, iOS support, or both, reach for the `--android` and/or `--ios` flags.

Once that's installed, the command below creates a plugin project:

- npm

```sh
npx @tauri-apps/cli plugin new [name]
```

That sets the plugin up inside the `tauri-plugin-[name]` directory, and according to whichever CLI flags you passed, the project that results takes this shape:

```plaintext
. tauri-plugin-[name]/
├── src/                - Rust code
│ ├── commands.rs       - Defines the commands the webview can use
| ├── desktop.rs        - Desktop implementation
| ├── error.rs          - Default error type to use in returned results
│ ├── lib.rs            - Re-exports appropriate implementation, setup state...
│ ├── mobile.rs         - Mobile implementation
│ └── models.rs         - Shared structs
├── permissions/        - This will host (generated) permission files for commands
├── android             - Android library
├── ios                 - Swift package
├── guest-js            - Source code of the JavaScript API bindings
├── dist-js             - Transpiled assets from guest-js
├── Cargo.toml          - Cargo crate metadata
└── package.json        - NPM package metadata
```

When a plugin already exists and you want to give it Android or iOS capabilities, the `plugin android add` and `plugin ios add` commands will scaffold the mobile library projects for you and walk you through the edits that are required.

### Mobile Plugin Development

Native mobile code authored in Kotlin (or Java) and in Swift can be executed by plugins. By default, the plugin template ships with both a Kotlin-based Android library project and a Swift package. Bundled in is a sample mobile command that demonstrates how Rust code can kick off its execution.

For a deeper look at building plugins aimed at mobile, consult the [Mobile Plugin Development guide](https://v2.tauri.app/develop/plugins/develop-mobile/).
