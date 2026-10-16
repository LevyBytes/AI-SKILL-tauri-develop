# Sidecar argument allowlists

Open when configuring fixed or validated sidecar arguments and passing them from Rust or JavaScript.

[Topic context](sidecar-execution-argument-permissions.md)

### Passing arguments

Handing arguments to Sidecar commands works the same way it does for a regular [Command](https://doc.rust-lang.org/std/process/struct.Command.html).

An argument comes in one of two flavors: **static** (for instance `-o` or `serve`) or **dynamic** (for instance `<file_path>` or `localhost:<PORT>`). Setting the value to `true` lets any argument through to the command, whereas `false` blocks all of them. When you supply neither `true` nor `false`, you spell out the arguments in the precise sequence you intend to call them. A static argument is written verbatim, while a dynamic one can be expressed through a regular expression.

To begin, declare in `src-tauri/capabilities/default.json` the arguments that must reach the sidecar command:

src-tauri/capabilities/default.json

```json
{
  "$schema": "../gen/schemas/desktop-schema.json",
  "identifier": "default",
  "description": "Capability for the main window",
  "windows": ["main"],
  "permissions": [
    "core:default",
    {
      "identifier": "shell:allow-execute",
      "allow": [
        {
          "args": [
            "arg1",
            "-a",
            "--arg2",
            {
              "validator": "\\S+"
            }
          ],
          "name": "binaries/my-sidecar",
          "sidecar": true
        }
      ]
    }
  ]
}
```

After that, invoking the sidecar command is just a matter of handing **all** the arguments over as an array.

Within Rust:

```rust
use tauri_plugin_shell::ShellExt;
#[tauri::command]
async fn call_my_sidecar(app: tauri::AppHandle) {
  let sidecar_command = app
    .shell()
    .sidecar("my-sidecar")
    .unwrap()
    .args(["arg1", "-a", "--arg2", "any-string-that-matches-the-validator"]);
  let (mut _rx, mut _child) = sidecar_command.spawn().unwrap();
}
```

Within JavaScript:

```javascript
import { Command } from '@tauri-apps/plugin-shell';
// notice that the args array matches EXACTLY what is specified in `capabilities/default.json`.
const command = Command.sidecar('binaries/my-sidecar', [
  'arg1',
  '-a',
  '--arg2',
  'any-string-that-matches-the-validator',
]);
const output = await command.execute();
```

---
