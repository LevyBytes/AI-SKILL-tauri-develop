# Rust global and webview listener registration

[Rust commands and events](rust-commands-events-index.md) / [Previous source section](frontend-to-rust-listener-lifecycle.md)

Context: Tauri v2: Rust global and webview listener registration. Read rust-event-listener-cleanup.md for explicit unlisten and once subscription cleanup. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-listening-to-events-on-rust"></a>

#### Listening to Events on Rust

Listeners that you register in Rust receive both global and webview-specific events as well.

-

Listening to global events

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

Listening to webview-specific events

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

[Rust event listener cleanup](rust-event-listener-cleanup.md) - Open when ending Rust event subscriptions with unlisten or registering a one-shot listener with once.
