# Resource path resolution and Android paths

[Topic context](resources-sidecars-index.md) / [Previous source section](resource-bundle-configuration.md) / [Next source section](resource-reading-access-permissions.md)

Context: Tauri v2: Resource path resolution and Android paths. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-resolve-resource-file-paths"></a> <a id="source-path-syntax"></a> <a id="source-android"></a>

### Resolve resource file paths

Rather than working out a resource file's path by hand, lean on the APIs listed here to resolve it for you

- Rust
- JavaScript

Over in Rust, obtain a [`PathResolver`](https://docs.rs/tauri/latest/tauri/path/struct.PathResolver.html) instance — both [`App`](https://docs.rs/tauri/latest/tauri/struct.App.html) and [`AppHandle`](https://docs.rs/tauri/latest/tauri/struct.AppHandle.html) can hand you one — and then invoke [`PathResolver::resolve`](https://docs.rs/tauri/latest/tauri/path/struct.PathResolver.html#method.resolve):

```rust
tauri::Builder::default()
  .setup(|app| {
    let resource_path = app.path().resolve("lang/de.json", BaseDirectory::Resource)?;
    Ok(())
  })
```

And to do the same within a command:

```rust
#[tauri::command]
fn hello(handle: tauri::AppHandle) {
  let resource_path = handle.path().resolve("lang/de.json", BaseDirectory::Resource)?;
}
```

From JavaScript, reach for [`resolveResource`](https://tauri.app/reference/javascript/api/namespacepath/#resolveresource) to work the path out:

```javascript
import { resolveResource } from '@tauri-apps/api/path';
const resourcePath = await resolveResource('lang/de.json');
```

### Path syntax

Whatever path you hand these API calls may be an ordinary relative one such as `folder/json_file.json`, which maps to `$RESOURCE/folder/json_file.json`, or something like `../relative/folder/toml_file.toml`, which maps to `$RESOURCE/_up_/relative/folder/toml_file.toml`; the rules these APIs follow are identical to the ones you'd apply writing `tauri.conf.json > bundle > resources`. For instance:

tauri.conf.json

```json
{
  "bundle": {
    "resources": ["folder/json_file.json", "../relative/folder/toml_file.toml"]
  }
}
```

```rust
let json_path = app.path().resolve("folder/json_file.json", BaseDirectory::Resource)?;
let toml_path = app.path().resolve("../relative/folder/toml_file.toml", BaseDirectory::Resource)?;
```

### Android

At present, resources live inside the APK as assets, which means what those APIs give back are not ordinary file system paths; instead Tauri prepends the dedicated URI prefix `asset://localhost/`, and that form works alongside the [`fs` plugin](https://v2.tauri.app/plugin/file-system/). With it in hand, read the files via [`FsExt::fs`](https://docs.rs/tauri-plugin-fs/latest/tauri_plugin_fs/trait.FsExt.html#tymethod.fs) in this fashion:

```rust
let resource_path = app.path().resolve("lang/de.json", BaseDirectory::Resource).unwrap();
let json = app.fs().read_to_string(&resource_path);
```

Should you need — or simply prefer — the resource files sitting on an actual file system, hand-copy their contents out by way of the [`fs` plugin](https://v2.tauri.app/plugin/file-system/)

