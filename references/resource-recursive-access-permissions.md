# Recursive resource permissions

Open when granting recursive resource access or denying specific files to filesystem and opener plugins.

[Topic context](resource-reading-access-permissions.md)

### Permissions

Because the list form swaps `../` for `_up_` in relative paths and swaps the root for `_root_` in absolute ones, the affected files end up nested in sub folders within the resource directory; to clear those paths under Tauri's [permission system](https://v2.tauri.app/security/capabilities/), apply `$RESOURCE/**/*`, which grants recursive access to them

### Examples

Given a file bundled in the following way:

tauri.conf.json

```json
{
  "bundle": {
    "resources": ["../relative/path/to/jsonfile.json"]
  }
}
```

Accessing it by way of the [`fs` plugin](https://v2.tauri.app/plugin/file-system/):

src-tauri/capabilities/default.json

```json
{
  "$schema": "../gen/schemas/desktop-schema.json",
  "identifier": "main-capability",
  "description": "Capability for the main window",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "fs:allow-stat",
    "fs:allow-read-text-file",
    "fs:allow-resource-read-recursive",
    {
      "identifier": "fs:scope",
      "allow": ["$RESOURCE/**/*"],
      "deny": ["$RESOURCE/secret.txt"]
    }
  ]
}
```

Reaching it through the [`opener` plugin](https://v2.tauri.app/plugin/opener/) instead:

src-tauri/capabilities/default.json

```json
{
  "$schema": "../gen/schemas/desktop-schema.json",
  "identifier": "main-capability",
  "description": "Capability for the main window",
  "windows": ["main"],
  "permissions": [
    "core:default",
    {
      "identifier": "opener:allow-open-path",
      "allow": [
        {
          "path": "$RESOURCE/**/*"
        }
      ]
    }
  ]
}
```

---
