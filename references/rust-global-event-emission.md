# Emit global Rust events

Open when choosing event delivery and emitting global events from Rust.

[Topic context](rust-event-emission.md)

# Calling Frontend

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

### Calling the Frontend from Rust

The notes below walk through the ways your Rust layer can reach into the app's frontend. For the reverse direction — driving Rust from the UI — refer to [Calling Rust from the Frontend](https://v2.tauri.app/develop/calling-rust/).

From the Rust portion of a Tauri app you can reach the frontend in three ways: through Tauri's event system, by way of channels, or by running JavaScript directly.

### Event System

Bundled with Tauri is a lightweight event system that lets Rust and the frontend talk to each other in both directions.

This mechanism suits cases where only modest amounts of data get streamed, or where you want a many-to-many producer/consumer arrangement — a push notification feed, for instance.

If you need very low latency or heavy throughput, the event system is the wrong tool; turn instead to the channels section, whose design is tuned for streaming.

A Tauri event differs from a Tauri command in several respects: events carry no strong typing, their payloads are always JSON strings (which makes them ill-suited to larger messages), and the [capabilities](https://v2.tauri.app/security/capabilities/) system — which would let you finely govern event data and channels — does not apply to them.

Both the [AppHandle](https://docs.rs/tauri/2.0.0/tauri/struct.AppHandle.html) and [WebviewWindow](https://docs.rs/tauri/2.0.0/tauri/webview/struct.WebviewWindow.html) types provide the event-system traits [Listener](https://docs.rs/tauri/2.0.0/tauri/trait.Listener.html) and [Emitter](https://docs.rs/tauri/2.0.0/tauri/trait.Emitter.html).

An event comes in two flavors: global ones reach every listener, while webview-specific ones go only to the webview whose label matches.

### Global Events

Firing a global event is done through the [Emitter#emit](https://docs.rs/tauri/2.0.0/tauri/trait.Emitter.html#tymethod.emit) function:

src-tauri/src/lib.rs

```rust
use tauri::{AppHandle, Emitter};

#[tauri::command]
fn download(app: AppHandle, url: String) {
  app.emit("download-started", &url).unwrap();
  for progress in [1, 15, 50, 80, 100] {
    app.emit("download-progress", progress).unwrap();
  }
  app.emit("download-finished", &url).unwrap();
}
```
