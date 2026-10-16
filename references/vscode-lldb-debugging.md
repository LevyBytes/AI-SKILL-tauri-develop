# Debug with VS Code LLDB

Open when configuring launch.json for the VS Code LLDB debugger.

[Topic context](vscode-tauri-debugging.md)

# Vscode

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

### Debug in VS Code

The steps below show how to configure VS Code so you can debug the [Core Process of your Tauri app](https://v2.tauri.app/concept/process-model/#the-core-process).

### All platforms with vscode-lldb extension

### Prerequisites

Add the [`vscode-lldb`](https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb) extension to your editor.

### Configure launch.json

Make a `.vscode/launch.json` file and drop the JSON shown below inside it:

.vscode/launch.json

```jsonc
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "lldb",
      "request": "launch",
      "name": "Tauri Development Debug",
      "cargo": {
        "args": [
          "build",
          "--manifest-path=./src-tauri/Cargo.toml",
          "--no-default-features"
        ]
      },
      // task for the `beforeDevCommand` if used, must be configured in `.vscode/tasks.json`
      "preLaunchTask": "ui:dev"
    },
    {
      "type": "lldb",
      "request": "launch",
      "name": "Tauri Production Debug",
      "cargo": {
        "args": ["build", "--release", "--manifest-path=./src-tauri/Cargo.toml"]
      },
      // task for the `beforeBuildCommand` if used, must be configured in `.vscode/tasks.json`
      "preLaunchTask": "ui:build"
    }
  ]
}
```

It invokes `cargo` on its own to compile the Rust binary and run it under either the development or production profile.

Keep in mind that the Tauri CLI is bypassed here, so anything specific to that CLI will not run. You therefore have to run the `beforeDevCommand` and `beforeBuildCommand` scripts yourself ahead of time, or wire them up through the `preLaunchTask` field. The sample `.vscode/tasks.json` shown next defines a pair of tasks: a `beforeDevCommand` task that launches a development server, plus one covering `beforeBuildCommand`:

.vscode/tasks.json

```jsonc
{
  // See https://go.microsoft.com/fwlink/?LinkId=733558
  // for the documentation about the tasks.json format
  "version": "2.0.0",
  "tasks": [
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
      "label": "ui:build",
      "type": "shell",
      // change this to your `beforeBuildCommand`:
      "command": "yarn",
      "args": ["build"]
    }
  ]
}
```

At this point you can drop breakpoints into `src-tauri/src/main.rs` (or whichever Rust source you like) and kick off a debug session with `F5`.
