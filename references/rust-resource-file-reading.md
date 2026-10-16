# Read resource files in Rust

Open when reading bundled JSON resources from Rust setup or a command.

[Topic context](resource-reading-access-permissions.md)

### Rust

From the Rust end, you'll want a [`PathResolver`](https://docs.rs/tauri/latest/tauri/path/struct.PathResolver.html) instance, which either [`App`](https://docs.rs/tauri/latest/tauri/struct.App.html) or [`AppHandle`](https://docs.rs/tauri/latest/tauri/struct.AppHandle.html) is able to provide:

```rust
use tauri::{Manager, path::BaseDirectory};

// Insert this setup callback in the application's Builder chain.
tauri::Builder::default()
  .setup(|app| {
    let resource_path = app.path().resolve("lang/de.json", BaseDirectory::Resource)?;
    let json = std::fs::read_to_string(&resource_path)?;
    let lang_de: serde_json::Value = serde_json::from_str(&json)?;
    println!("{}", lang_de["hello"].as_str().ok_or("missing hello string")?);
    Ok(())
  })
```

```rust
use tauri::{Manager, path::BaseDirectory};

#[tauri::command]
fn hello(handle: tauri::AppHandle) -> Result<String, String> {
    let path = handle.path().resolve("lang/de.json", BaseDirectory::Resource)
        .map_err(|error| error.to_string())?;
    let json = std::fs::read_to_string(path).map_err(|error| error.to_string())?;
    let values: serde_json::Value = serde_json::from_str(&json)
        .map_err(|error| error.to_string())?;
    values.get("hello").and_then(|value| value.as_str())
        .map(str::to_owned).ok_or_else(|| "missing hello string".to_owned())
}
```

Both examples use the [bundled JSON file layout](resource-example-file-layout.md) and the serde_json dependency. Register hello in the invoke handler. On Android, read the asset through the filesystem plugin; std::fs cannot read the Android asset URI.
