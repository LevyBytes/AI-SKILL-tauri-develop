# Command and global scopes

Open when reading command-specific and global permission scopes in a plugin.

[Topic context](plugin-permissions-scope-schemas.md)

###### Command Scope

Anyone consuming your plugin is able to set a scope for one particular command within their capability file (refer to the [documentation](https://v2.tauri.app/reference/acl/scope/)). To read back that command-specific scope, use the [`tauri::ipc::CommandScope`](https://docs.rs/tauri/2.0.0/tauri/ipc/struct.CommandScope.html) struct:

src/commands.rs

```rust
use tauri::ipc::CommandScope;
use crate::scope::Entry;

async fn spawn<R: tauri::Runtime>(app: tauri::AppHandle<R>, command_scope: CommandScope<'_, Entry>) -> Result<()> {
  let allowed = command_scope.allows();
  let denied = command_scope.denies();
  todo!()
}
```

###### Global Scope

Should a permission name no commands at all to grant or refuse, it counts as a scope permission, and its only job is to lay out a global scope for your plugin:

permissions/spawn-node.toml

```toml
[[permission]]
identifier = "allow-spawn-node"
description = "This scope permits spawning the `node` binary."

[[permission.scope.allow]]
binary = "node"
```

Reading the global scope is done by way of the [`tauri::ipc::GlobalScope`](https://docs.rs/tauri/2.0.0/tauri/ipc/struct.GlobalScope.html) struct:

src/commands.rs

```rust
use tauri::ipc::GlobalScope;
use crate::scope::Entry;

async fn spawn<R: tauri::Runtime>(app: tauri::AppHandle<R>, scope: GlobalScope<'_, Entry>) -> Result<()> {
  let allowed = scope.allows();
  let denied = scope.denies();
  todo!()
}
```

Use the Entry type defined with [command permissions](plugin-command-permissions.md).
