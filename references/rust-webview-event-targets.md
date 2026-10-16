# Target webview events

Open when emitting to one webview or filtering event targets.

[Topic context](rust-event-emission.md)

### Webview Event

When you want to reach a listener that one particular webview registered, reach for the [Emitter#emit_to](https://docs.rs/tauri/2.0.0/tauri/trait.Emitter.html#tymethod.emit_to) function:

src-tauri/src/lib.rs

```rust
use tauri::{AppHandle, Emitter};

#[tauri::command]
fn login(app: AppHandle, user: String, password: String) {
  let authenticated = user == "tauri-apps" && password == "tauri";
  let result = if authenticated { "loggedIn" } else { "invalidCredentials" };
  app.emit_to("login", "login-result", result).unwrap();
}
```

You may also dispatch an event to a set of webviews via [Emitter#emit_filter](https://docs.rs/tauri/2.0.0/tauri/trait.Emitter.html#tymethod.emit_filter). The snippet below sends an open-file event to the main and file-viewer webviews:

src-tauri/src/lib.rs

```rust
use tauri::{AppHandle, Emitter, EventTarget};

#[tauri::command]
fn open_file(app: AppHandle, path: std::path::PathBuf) {
  app.emit_filter("open-file", path, |target| match target {
    EventTarget::WebviewWindow { label } => label == "main" || label == "file-viewer",
    _ => false,
  }).unwrap();
}
```
