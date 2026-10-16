# VS Code Windows native debugger

Open when configuring the Windows Visual Studio debugger and build tasks in VS Code.

[Topic context](vscode-tauri-debugging.md)

### With Visual Studio Windows Debugger on Windows

Available solely on Windows, the Visual Studio Windows Debugger tends to outpace [`vscode-lldb`](https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb) and handles certain Rust constructs, enums among them, more gracefully.

### Prerequisites

Grab the [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) extension, then work through [https://code.visualstudio.com/docs/cpp/config-msvc#\_prerequisites](https://code.visualstudio.com/docs/cpp/config-msvc#_prerequisites) to get the Visual Studio Windows Debugger set up.

### Configure launch.json and tasks.json

.vscode/launch.json

```jsonc
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch App Debug",
      "type": "cppvsdbg",
      "request": "launch",
      // change the exe name to your actual exe name
      // (to debug release builds, change `target/debug` to `release/debug`)
      "program": "${workspaceRoot}/src-tauri/target/debug/your-app-name-here.exe",
      "cwd": "${workspaceRoot}",
      "preLaunchTask": "ui:dev"
    }
  ]
}
```

As before, the Tauri CLI plays no part here, so any CLI-only capabilities stay dormant. Your `tasks.json` mirrors the `lldb` version, the one difference being that you add a config group and point the `preLaunchTask` in `launch.json` at it whenever you want a compile to happen automatically before each launch.

What follows demonstrates bundling a dev server (the stand-in for `beforeDevCommand`) together with the build step (`cargo build`) into one group; to put it to work, set the `preLaunchTask` value inside `launch.json` to `dev` (or whatever label you gave that group).

.vscode/tasks.json

```jsonc
{
  // See https://go.microsoft.com/fwlink/?LinkId=733558
  // for the documentation about the tasks.json format
  "version": "2.0.0",
  "tasks": [
    {
      "label": "build:debug",
      "type": "cargo",
      "command": "build",
      "options": {
        "cwd": "${workspaceRoot}/src-tauri"
      }
    },
    {
      "label": "ui:dev",
      "type": "shell",
      // `dev` keeps running in the background
      // ideally you should also configure a `problemMatcher`
      // see https://code.visualstudio.com/docs/editor/tasks#_can-a-background-task-be-used-as-a-prelaunchtask-in-launchjson
      "isBackground": true,
      // change this to your `beforeDevCommand`:
      "command": "yarn",
      "args": ["dev"]
    },
    {
      "label": "dev",
      "dependsOn": ["build:debug", "ui:dev"],
      "group": {
        "kind": "build"
      }
    }
  ]
}
```

---
