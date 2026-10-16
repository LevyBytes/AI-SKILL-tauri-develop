# Rust listeners and subscription lifetime

[Topic context](rust-frontend-communication-index.md) / [Previous source section](rust-to-frontend-listener-lifecycle.md) / [Next source section](frontend-channels-javascript-evaluation.md)

Context: Tauri v2: Rust listeners and subscription lifetime. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-listening-to-events-on-rust"></a>

#### Listening to Events on Rust

Listeners that you register on the Rust side receive global and webview-specific events too.

-

Subscribing to global events

src-tauri/src/lib.rs

```rust
use tauri::Listener;

#[derive(serde::Deserialize)]
struct DownloadStarted { url: String }

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
  tauri::Builder::default()
    .setup(|app| {
      app.listen("download-started", |event| {
        if let Ok(payload) = serde_json::from_str::<DownloadStarted>(event.payload()) {
          println!("downloading {}", payload.url);
        }
      });
      Ok(())
    })
    .run(tauri::generate_context!())
    .expect("error while running tauri application");
}
```

-

Subscribing to webview-scoped events

src-tauri/src/lib.rs

```rust
use tauri::{Listener, Manager};

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
  tauri::Builder::default()
    .setup(|app| {
      let webview = app.get_webview_window("main").unwrap();
      webview.listen("logged-in", |event| {
        let session_token = serde_json::from_str::<String>(event.payload()).unwrap();
        // save token..
      });
      Ok(())
    })
    .run(tauri::generate_context!())
    .expect("error while running tauri application");
}
```

A subscription made with `listen` persists for the whole run of the application. To end it, reach for the `unlisten` function:

```rust
// unlisten outside of the event handler scope:
let event_id = app.listen("download-started", |event| {});
app.unlisten(event_id);

// unlisten when some event criteria is matched
let handle = app.handle().clone();
app.listen("status-changed", move |event| {
  if serde_json::from_str::<String>(event.payload()).is_ok_and(|value| value == "ready") {
    handle.unlisten(event.id());
  }
});
```

There is likewise a convenience helper on this side for catching an event one solitary time:

```rust
app.once("ready", |event| {
  println!("app is ready");
});
```

Here the listener gets dropped the moment it fires for the first time.

