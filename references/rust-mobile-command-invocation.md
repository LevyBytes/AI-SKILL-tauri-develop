# Invoke mobile commands from Rust

Open when calling a mobile plugin command from its Rust wrapper.

[Topic context](mobile-command-argument-models.md)

Lean on the [`tauri::plugin::PluginHandle`](https://docs.rs/tauri/2.0.0/tauri/plugin/struct.PluginHandle.html) when you need to fire a mobile command from the Rust side:

```rust
use std::path::PathBuf;
use serde::{Deserialize, Serialize};
use tauri::Runtime;

#[derive(Serialize)]
#[serde(rename_all = "camelCase")]
pub struct CameraRequest {
  quality: usize,
  allow_edit: bool,
}

#[derive(Deserialize)]
pub struct Photo {
  path: PathBuf,
}


impl<R: Runtime> <plugin-name;pascal-case><R> {
  pub fn open_camera(&self, payload: CameraRequest) -> crate::Result<Photo> {
    self
      .0
      .run_mobile_plugin("openCamera", payload)
      .map_err(Into::into)
  }
}
```

The angle-bracket plugin name in the Rust template stands for your generated plugin wrapper type; substitute that type before compiling.
