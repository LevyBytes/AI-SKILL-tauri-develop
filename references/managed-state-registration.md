# Register managed state

Open when registering app-wide state and retrieving it from an application handle.

[Topic context](managed-state-mutex-selection.md)

# State Management, Updating Dependencies, Icons

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: State Management; Updating Dependencies; App Icons.

## State Management

### State Management

Most Tauri apps reach a point where they must remember some current condition or oversee the lifecycle of resources tied to the app. With the [`Manager`](https://docs.rs/tauri/latest/tauri/trait.Manager.html) API, Tauri gives you a straightforward path to hold that data and pull it back out whenever a command runs.

A minimal illustration follows:

```rust
use tauri::{Builder, Manager};

struct AppData {
  welcome_message: &'static str,
}

fn main() {
  Builder::default()
    .setup(|app| {
      app.manage(AppData {
        welcome_message: "Welcome to Tauri!",
      });
      Ok(())
    })
    .run(tauri::generate_context!())
    .unwrap();
}
```

Afterward, retrieving that stored value is possible from anything carrying the [`Manager`](https://docs.rs/tauri/latest/tauri/trait.Manager.html) trait — the [`App`](https://docs.rs/tauri/latest/tauri/struct.App.html) instance being one such example:

```rust
let data = app.state::<AppData>();
```

The Accessing State section below has additional details, such as how to reach state from within commands.
