# Register multiple commands

Open when registering several Rust commands in one invoke handler.

[Topic context](rust-commands-requests-registration.md)

### Creating Multiple Commands

An array of commands is what the `tauri::generate_handler!` macro expects. Registering several commands is not done by calling invoke_handler more than once — only the final such call counts. Every command has to go into one solitary call of `tauri::generate_handler!`.

src-tauri/src/lib.rs

```rust
#[tauri::command]
fn cmd_a() -> String {
  "Command a"
}
#[tauri::command]
fn cmd_b() -> String {
  "Command b"
}

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
  tauri::Builder::default()
    .invoke_handler(tauri::generate_handler![cmd_a, cmd_b])
    .run(tauri::generate_context!())
    .expect("error while running tauri application");
}
```
