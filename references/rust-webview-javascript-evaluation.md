# Evaluate JavaScript from Rust

Open when evaluating JavaScript in a webview from Rust.

[Topic context](frontend-channels-javascript-evaluation.md)

### Evaluating JavaScript

For running arbitrary JavaScript straight inside the webview context, the [`WebviewWindow#eval`](https://docs.rs/tauri/2.0.0/tauri/webview/struct.WebviewWindow.html#method.eval) function is what you want:

src-tauri/src/lib.rs

```rust
use tauri::Manager;

tauri::Builder::default()
  .setup(|app| {
    let webview = app.get_webview_window("main").unwrap();
    webview.eval("console.log('hello from Rust')")?;
    Ok(())
  })
```

When the script you're evaluating grows more involved and has to draw on data from Rust objects, the [serialize-to-javascript](https://docs.rs/serialize-to-javascript/latest/serialize_to_javascript/) crate is the recommended choice.

---
