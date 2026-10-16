# Plugin setup and background work

Open when initializing managed state and background work in plugin setup.

[Topic context](plugin-setup-lifecycle-hooks.md)

### Lifecycle Events

A plugin is able to attach itself to a number of lifecycle events:

- setup: the plugin is undergoing initialization
- on_navigation: the web view is trying to carry out navigation
- on_webview_ready: a new window is in the process of being created
- on_event: events from the event loop
- on_drop: the plugin is being torn down

Mobile plugins have further [lifecycle events for mobile plugins](https://v2.tauri.app/develop/plugins/develop-mobile/#lifecycle-events).

### setup

- **When**: the plugin is undergoing initialization
- **Why**: hook up mobile plugins, handle state, kick off background work
  src/lib.rs

```rust
use tauri::{Emitter, Manager, plugin::Builder};
use std::{collections::HashMap, sync::Mutex, time::Duration};

struct DummyStore(Mutex<HashMap<String, String>>);

Builder::new("<plugin-name>")
  .setup(|app, api| {
    app.manage(DummyStore(Default::default()));

    let app_ = app.clone();
    std::thread::spawn(move || {
      loop {
        app_.emit("tick", ());
        std::thread::sleep(Duration::from_secs(1));
      }
    });

    Ok(())
  })
```
