# CrabNebula DevTools setup

[Topic context](debugging-tools-index.md) / [Previous source section](debug-builds-production-inspector.md) / [Next source section](linux-graphics-troubleshooting.md)

Context: Tauri v2: CrabNebula DevTools setup. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-crabnebula-devtools"></a> <a id="source-crabnebula-devtools-1"></a>

## CrabNebula DevTools

### CrabNebula DevTools

Through its collaboration with the Tauri project, [CrabNebula](https://crabnebula.dev/) offers a no-cost [DevTools](https://crabnebula.dev/devtools/) application for Tauri. With it you can instrument your Tauri app: it records the bundled assets, the Tauri configuration file, plus logs and spans, then serves a web frontend that renders all of that data live as it happens.

The CrabNebula DevTools let you examine every log event your app emits (dependency logs included), measure how your command invocations perform alongside broader Tauri API activity, and work through a dedicated view for Tauri events and commands that exposes their payload, responses, nested logs, and execution spans.

Switching on the CrabNebula DevTools begins with adding the devtools crate:

```sh
cargo add tauri-plugin-devtools@2.0.0
```

Then bring the plugin up at the earliest opportunity inside your main function:

```rust
fn main() {
    // This should be called as early in the execution of the app as possible
    #[cfg(debug_assertions)] // only enable instrumentation in development builds
    let devtools = tauri_plugin_devtools::init();

    let mut builder = tauri::Builder::default();

    #[cfg(debug_assertions)]
    {
        builder = builder.plugin(devtools);
    }

    builder
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

After that, start your app the usual way; assuming the configuration is right, devtools emits this message:

(Figure: DevTools message on terminal)

Further detail lives in the [CrabNebula DevTools](https://docs.crabnebula.dev/devtools/get-started/) documentation.

---

