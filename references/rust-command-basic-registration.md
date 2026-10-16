# Register and invoke commands

Open when defining a Rust command, registering its handler and invoking it from JavaScript.

[Topic context](rust-command-registration-wasm.md)

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

### Calling Rust from the Frontend

Here you will find walkthroughs covering the ways your application frontend can talk to your Rust code. For the reverse direction — driving the frontend from within Rust — consult [Calling the Frontend from Rust](https://v2.tauri.app/develop/calling-frontend/).

For reaching Rust functions with type safety, Tauri offers a command primitive; it pairs that with a more flexible, dynamic event system.

### Commands

For invoking Rust functions out of your web app, Tauri ships a `command` system that is straightforward but capable. A command may take in arguments and hand back values; it can likewise surface errors and run as `async`.

### Basic Example

You declare commands within the `src-tauri/src/lib.rs` file. Defining one is as easy as writing a function and tagging it with `#[tauri::command]`:

src-tauri/src/lib.rs

```rust
#[tauri::command]
fn my_custom_command() {
  println!("I was invoked from JavaScript!");
}
```

Next, hand the builder function a roster of every command you have, in this fashion:

src-tauri/src/lib.rs

```rust
#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
  tauri::Builder::default()
    .invoke_handler(tauri::generate_handler![my_custom_command])
    .run(tauri::generate_context!())
    .expect("error while running tauri application");
}
```

With that done, your JavaScript code is free to call the command:

```javascript
import { invoke } from '@tauri-apps/api/core';

// Invoke the registered command using the npm API.
invoke('my_custom_command');
```

For an app with `app.withGlobalTauri: true`, use this alternative without the npm import:

```javascript
const invoke = window.__TAURI__.core.invoke;
invoke('my_custom_command');
```
