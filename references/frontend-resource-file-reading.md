# Read resource files from the frontend

Open when reading bundled JSON through the filesystem plugin and its capability permissions.

[Topic context](resource-reading-access-permissions.md)

### JavaScript

Over on the JavaScript side, you have two options: either define a command resembling the one shown above and trigger it via `await invoke('hello')`, or get at the files straight through the [`fs` plugin](https://v2.tauri.app/plugin/file-system/).

Going the [`fs` plugin](https://v2.tauri.app/plugin/file-system/) route means that, beyond the [basic setup](https://v2.tauri.app/plugin/file-system/#setup), you must also set up the access control list so the plugin APIs you depend on are turned on, together with the permissions required to reach the `$RESOURCE` folder:

src-tauri/capabilities/default.json

```json
{
  "$schema": "../gen/schemas/desktop-schema.json",
  "identifier": "main-capability",
  "description": "Capability for the main window",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "fs:allow-read-text-file",
    "fs:allow-resource-read-recursive"
  ]
}
```

```javascript
import { resolveResource } from '@tauri-apps/api/path';
import { readTextFile } from '@tauri-apps/plugin-fs';

const resourcePath = await resolveResource('lang/de.json');
const langDe = JSON.parse(await readTextFile(resourcePath));
console.log(langDe.hello); // This will print 'Guten Tag!' to the devtools console
```

Use the [bundled JSON file layout](resource-example-file-layout.md); the Rust command alternative is in [Rust resource reading](rust-resource-file-reading.md).
