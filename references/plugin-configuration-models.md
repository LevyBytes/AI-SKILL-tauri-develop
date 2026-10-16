# Plugin configuration models

Open when defining plugin configuration and deserializing it in Rust.

[Topic context](plugin-initialization-naming-configuration.md)

### Plugin Configuration

Within whichever Tauri application makes use of the plugin, you declare the plugin configuration inside `tauri.conf.json`, where `plugin-name` stands for the plugin's name. Merge this fragment into your existing configuration:

```json
{
  "plugins": {
    "plugin-name": { "timeout": 30 }
  }
}
```

You attach the plugin's configuration to the `Builder`, and it gets parsed while the app runs. Below is a sample showing the `Config` struct put to work to describe the plugin configuration:

src/lib.rs

```rust
use serde::Deserialize;
use tauri::{
    plugin::{Builder, TauriPlugin},
    Runtime,
};

// Define the plugin config
#[derive(Deserialize)]
pub struct Config {
  timeout: usize,
}

pub fn init<R: Runtime>() -> TauriPlugin<R, Config> {
  // Make the plugin config optional
  // by using `Builder::<R, Option<Config>>` instead
  Builder::<R, Config>::new("<plugin-name>")
    .setup(|app, api| {
      let timeout = api.config().timeout;
      Ok(())
    })
    .build()
}
```
