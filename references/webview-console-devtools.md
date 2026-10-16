# Webview console and devtools

Open when opening the WebView inspector programmatically and reading frontend console output.

[Topic context](rust-webview-development-diagnostics.md)

### WebView Console

Within the WebView, right-click and pick `Inspect Element`. Doing so launches a web-inspector much like the Chrome or Firefox developer tools you already know. The inspector can also be summoned with the `Ctrl + Shift + i` shortcut on Linux and Windows, or `Command + Option + i` over on macOS.

Which inspector appears depends on the platform: the webkit2gtk WebInspector shows up under Linux, Safari’s inspector under macOS, and the Microsoft Edge DevTools under Windows.

### Opening Devtools Programmatically

Toggling whether the inspector window is shown is done through the [`WebviewWindow::open_devtools`](https://docs.rs/tauri/2.0.0/tauri/webview/struct.WebviewWindow.html#method.open_devtools) and [`WebviewWindow::close_devtools`](https://docs.rs/tauri/2.0.0/tauri/webview/struct.WebviewWindow.html#method.close_devtools) functions:

```rust
tauri::Builder::default()
  .setup(|app| {
    #[cfg(debug_assertions)] // only include this code on debug builds
    {
      let window = app.get_webview_window("main").unwrap();
      window.open_devtools();
      window.close_devtools();
    }
    Ok(())
  });
```
