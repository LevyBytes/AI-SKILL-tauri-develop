# Command channels, window, app handle and state

[Rust commands and events](rust-commands-events-index.md) / [Previous source section](async-rust-command-invocation.md) / [Next source section](rust-commands-requests-registration.md)

Context: Tauri v2: Command channels, window, app handle and state. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-channels"></a> <a id="source-accessing-the-webviewwindow-in-commands"></a> <a id="source-accessing-an-apphandle-in-commands"></a> <a id="source-accessing-managed-state"></a>

### Channels

When you need to stream data to the frontend — streamed HTTP responses, for instance — the Tauri channel is the advised tool. In the example below, a file is read and the frontend gets progress updates in 4096-byte chunks:

```rust
use tokio::io::AsyncReadExt;

#[tauri::command]
async fn load_image(path: std::path::PathBuf, reader: tauri::ipc::Channel<&[u8]>) {
  // for simplicity this example does not include error handling
  let mut file = tokio::fs::File::open(path).await.unwrap();

  let mut chunk = vec![0; 4096];

  loop {
    let len = file.read(&mut chunk).await.unwrap();
    if len == 0 {
      // Length of zero means end of file.
      break;
    }
    reader.send(&chunk[..len]).unwrap();
  }
}
```

Further detail lives in the [channels documentation](https://v2.tauri.app/develop/calling-frontend/#channels).

### Accessing the WebviewWindow in Commands

A command is able to get at the `WebviewWindow` instance that triggered the message:

src-tauri/src/lib.rs

```rust
#[tauri::command]
async fn my_custom_command(webview_window: tauri::WebviewWindow) {
  println!("WebviewWindow: {}", webview_window.label());
}
```

### Accessing an AppHandle in Commands

Reaching an `AppHandle` instance is also possible from a command:

src-tauri/src/lib.rs

```rust
use tauri::Manager;
use tauri_plugin_global_shortcut::GlobalShortcutExt;

#[tauri::command]
fn my_custom_command(app_handle: tauri::AppHandle) -> Result<(), String> {
  let _app_dir = app_handle.path().app_data_dir().map_err(|e| e.to_string())?;
  app_handle.global_shortcut().on_shortcut("CTRL+U", |_app, _shortcut, _event| {})
    .map_err(|e| e.to_string())?;
  Ok(())
}
```

### Accessing Managed State

Through the `manage` function on `tauri::Builder`, Tauri takes care of state for you. A command then gets at that state by way of `tauri::State`:

src-tauri/src/lib.rs

```rust
struct MyState(String);

#[tauri::command]
fn my_custom_command(state: tauri::State<MyState>) {
  assert_eq!(state.0 == "some state value", true);
}

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
  tauri::Builder::default()
    .manage(MyState("some state value".into()))
    .invoke_handler(tauri::generate_handler![my_custom_command])
    .run(tauri::generate_context!())
    .expect("error while running tauri application");
}
```


The AppHandle shortcut example is desktop-only and requires the global-shortcut plugin dependency and Builder initialization described in [the official plugin setup](https://v2.tauri.app/plugin/global-shortcut/).
