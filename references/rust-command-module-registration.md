# Register commands from a module

Open when keeping public command functions in a separate Rust module.

[Topic context](rust-command-registration-wasm.md)

#### Defining Commands in a Separate Module

When your app declares many components, or when those components fall into natural groups, you can keep commands in their own module rather than swelling the `lib.rs` file.

By way of illustration, let us put a command inside the `src-tauri/src/commands.rs` file:

src-tauri/src/commands.rs

```rust
#[tauri::command]
pub fn my_custom_command() {
  println!("I was invoked from JavaScript!");
}
```

Over in the `lib.rs` file, declare that module and supply the command roster to match;

src-tauri/src/lib.rs

```rust
mod commands;

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
  tauri::Builder::default()
    .invoke_handler(tauri::generate_handler![commands::my_custom_command])
    .run(tauri::generate_context!())
    .expect("error while running tauri application");
}
```

Observe how the `commands::` prefix appears in the roster; it spells out the complete path leading to the command function.

Because the command here is named `my_custom_command`, you keep invoking it from the frontend by running `invoke("my_custom_command")` — the `commands::` prefix is dropped.
