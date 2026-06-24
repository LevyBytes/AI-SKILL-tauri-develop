# Calling Rust (part 2)

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

A handler attached via `listen` stays active for as long as the program runs. When you want that subscription to end, call the `unlisten` function:

```
// unlisten outside of the event handler scope:let event_id = app.listen("download-started", |event| {});app.unlisten(event_id);
// unlisten when some event criteria is matchedlet handle = app.handle().clone();app.listen("status-changed", |event| {  if event.data == "ready" {    handle.unlisten(event.id);  }});
```

Tauri also ships a helper that subscribes to a single event and reacts only on its first occurrence:

```
app.once("ready", |event| {  println!("app is ready");});
```

Here the subscription tears itself down right after that initial firing.

For details on subscribing to and dispatching events directly from Rust, consult the [Rust Event System documentation](https://v2.tauri.app/develop/calling-frontend/#event-system).

---
