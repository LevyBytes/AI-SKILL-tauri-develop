# Plugin command permissions

Open when defining plugin permission files and the type used for permission scopes.

[Topic context](plugin-permissions-scope-schemas.md)

### Command Permissions

Out of the box, the frontend cannot reach your commands. Attempt to run any of them and you'll be met with a denied error rejection. For commands to genuinely be exposed, you additionally have to declare permissions that grant access to each one.

#### Permission Files

You express permissions as JSON or TOML files placed within the `permissions` directory. Any given file may declare a set of permissions, a set of permission sets, and the default permission for your plugin.

##### Permissions

What a permission spells out is the privileges held by your plugin commands. It is able to grant or refuse a set of commands and tie together scopes that are global as well as ones tied to particular commands.

permissions/start-server.toml

```toml
"$schema" = "schemas/schema.json"

[[permission]]
identifier = "allow-start-server"
description = "Enables the start_server command."
commands.allow = ["start_server"]

[[permission]]
identifier = "deny-start-server"
description = "Denies the start_server command."
commands.deny = ["start_server"]
```

##### Scope

By means of scopes, your plugin can place finer-grained limits on individual commands. Within each permission you may declare a set of scope objects, every one of which marks something as granted or refused, whether for a single command or across the whole plugin.

Below, let's lay out a sample struct meant to carry scope data covering the set of binaries that a `shell` plugin may spawn:

src/scope.rs

```rust
#[derive(Debug, schemars::JsonSchema)]
pub struct Entry {
    pub binary: String,
}
```
