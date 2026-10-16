# Generate plugin scope schemas

Open when sharing scope types with build.rs and generating permission schemas.

[Topic context](plugin-permissions-scope-schemas.md)

###### Schema

For the scope entry to produce a JSON schema, it leans on the `schemars` dependency, which lets those consuming the plugin understand the scope's format and get autocomplete inside their IDEs.

Setting up the schema starts with adding the dependency to your Cargo.toml file:

```toml
# we need to add schemars to both dependencies and build-dependencies because the scope.rs module is shared between the app code and build script
[dependencies]
schemars = "0.8"

[build-dependencies]
schemars = "0.8"
```

Inside your build script, drop in the code below:

build.rs

```rust
#[path = "src/scope.rs"]
mod scope;

const COMMANDS: &[&str] = &[];

fn main() {
    tauri_plugin::Builder::new(COMMANDS)
        .global_scope_schema(schemars::schema_for!(scope::Entry))
        .build();
}
```

Define src/scope.rs using the scope type in [command permissions](plugin-command-permissions.md).
