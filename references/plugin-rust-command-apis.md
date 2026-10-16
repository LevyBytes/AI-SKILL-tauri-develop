# Expose Rust APIs and commands

[Topic context](plugin-development-index.md) / [Previous source section](plugin-setup-lifecycle-hooks.md) / [Next source section](plugin-permissions-scope-schemas.md)

Context: Tauri v2: Expose Rust APIs and commands. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-exposing-rust-apis"></a> <a id="source-adding-commands"></a>

### Exposing Rust APIs

Whatever plugin APIs you declare in the project's `desktop.rs` and `mobile.rs` get handed to the user as a struct bearing the plugin's name in pascal case. At plugin setup time, one instance of that struct is built and held as state, which means users can pull it out whenever they like via a `Manager` instance (for example `AppHandle`, `App`, or` Window`) by way of the extension trait the plugin declares.

To illustrate, the [`global-shortcut plugin`](https://v2.tauri.app/plugin/global-shortcut/) declares a `GlobalShortcut` struct, and you reach it through the `global_shortcut` method on the `GlobalShortcutExt` trait:

src-tauri/src/lib.rs

```rust
use tauri_plugin_global_shortcut::GlobalShortcutExt;

tauri::Builder::default()
  .plugin(tauri_plugin_global_shortcut::Builder::new().build())
  .setup(|app| {
    app.global_shortcut().register("CTRL+U")?;
    Ok(())
  })
```

### Adding Commands

You declare commands in the `commands.rs` file, and they behave just like ordinary Tauri application commands. They can reach the AppHandle and Window instances straight away, get at state, and accept input in the very same fashion as application commands do. For more on Tauri commands, consult the [Commands guide](https://v2.tauri.app/develop/calling-rust/).

The following command demonstrates obtaining the `AppHandle` and `Window` instance through dependency injection, and it accepts a pair of input parameters (`on_progress` and `url`):

src/commands.rs

```rust
use tauri::{command, ipc::Channel, AppHandle, Runtime, Window};

#[command]
async fn upload<R: Runtime>(app: AppHandle<R>, window: Window<R>, on_progress: Channel, url: String) {
  // implement command logic here
  on_progress.send(100).unwrap();
}
```

So that the webview can reach the command, you have to wire it into the `invoke_handler()` call inside `lib.rs`:

src/lib.rs

```rust
Builder::new("<plugin-name>")
    .invoke_handler(tauri::generate_handler![commands::upload])
```

Declare a binding function within `webview-src/index.ts` so the people using your plugin can invoke the command from JavaScript without trouble:

```ts
import { invoke, Channel } from '@tauri-apps/api/core'

export async function upload(url: string, onProgressHandler: (progress: number) => void): Promise<void> {
  const onProgress = new Channel<number>()
  onProgress.onmessage = onProgressHandler
  await invoke('plugin:<plugin-name>|upload', { url, onProgress })
}
```

Make a point of compiling the TypeScript code before you try testing it.

