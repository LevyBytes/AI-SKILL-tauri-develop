# Plugin event and exit lifecycle

Open when handling event-loop notifications, saving state at exit or running plugin teardown.

[Topic context](plugin-setup-lifecycle-hooks.md)

### on_event

- **When**: events from the event loop
- **Why**: respond to core events like window events, menu events, and a requested application exit

This lifecycle hook lets you receive notification of every event loop [events](https://docs.rs/tauri/2.0.0/tauri/enum.RunEvent.html).

src/lib.rs

```rust
use std::{collections::HashMap, fs::write, sync::Mutex};
use tauri::{plugin::Builder, Manager, RunEvent};

struct DummyStore(Mutex<HashMap<String, String>>);

Builder::new("<plugin-name>")
  .setup(|app, _api| {
    app.manage(DummyStore(Default::default()));
    Ok(())
  })
  .on_event(|app, event| {
    match event {
      RunEvent::ExitRequested { api, .. } => {
        // user requested a window to be closed and there's no windows left

        // we can prevent the app from exiting:
        api.prevent_exit();
      }
      RunEvent::Exit => {
        // app is going to exit, you can cleanup here

        let store = app.state::<DummyStore>();
        write(
          app.path().app_local_data_dir().unwrap().join("store.json"),
          serde_json::to_string(&*store.0.lock().unwrap()).unwrap(),
        )
        .unwrap();
      }
      _ => {}
    }
  })
```

### on_drop

- **When**: the plugin is being torn down
- **Why**: run code once the plugin has been destroyed

For further detail, refer to [`Drop`](https://doc.rust-lang.org/std/ops/trait.Drop.html).

src/lib.rs

```rust
use tauri::plugin::Builder;

Builder::new("<plugin-name>")
  .on_drop(|app| {
    // plugin has been destroyed...
  })
```
