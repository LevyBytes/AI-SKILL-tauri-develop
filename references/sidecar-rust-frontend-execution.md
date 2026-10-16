# Run sidecars from Rust and the frontend

Open when spawning a bundled sidecar and enabling frontend execution permissions.

[Topic context](sidecar-execution-argument-permissions.md)

### Running it from Rust

In Rust, bring in the `tauri_plugin_shell::ShellExt` trait and invoke the `shell().sidecar()` function against the AppHandle:

```rust
use tauri_plugin_shell::ShellExt;
use tauri_plugin_shell::process::CommandEvent;
use tauri::Emitter;

let sidecar_command = app.shell().sidecar("my-sidecar").unwrap();
let (mut rx, mut child) = sidecar_command
  .spawn()
  .expect("Failed to spawn sidecar");

tauri::async_runtime::spawn(async move {
  // read events such as stdout
  while let Some(event) = rx.recv().await {
    if let CommandEvent::Stdout(line_bytes) = event {
      let line = String::from_utf8_lossy(&line_bytes);
      app
        .emit("message", Some(format!("'{}'", line)))
        .expect("failed to emit event");
      // write to stdin
      child.write("message from Rust\n".as_bytes()).unwrap();
    }
  }
});
```

Drop this code within a Tauri command for a convenient way to pass the AppHandle along, or stash a reference to the AppHandle inside the builder script so you can get at it from anywhere else in your application.

### Running it from JavaScript

To launch the sidecar, Tauri insists that you first authorize it to invoke the `execute` or `spawn` method on the child process. Grant that authorization by opening the file `<PROJECT ROOT>/src-tauri/capabilities/default.json` and dropping the block shown below into its permissions array. Remember to label your sidecar in line with the relative path noted earlier.

src-tauri/capabilities/default.json

```json
{
  "permissions": [
    "core:default",
    {
      "identifier": "shell:allow-execute",
      "allow": [
        {
          "name": "binaries/app",
          "sidecar": true
        }
      ]
    }
  ]
}
```

Within the JavaScript code, pull in the `Command` class from the `@tauri-apps/plugin-shell` module and call its `sidecar` static method.

```javascript
import { Command } from '@tauri-apps/plugin-shell';
const command = Command.sidecar('binaries/my-sidecar');
const output = await command.execute();
```
