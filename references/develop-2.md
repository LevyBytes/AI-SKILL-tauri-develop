# Resources, Sidecar

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Embedding Additional Files; Embedding External Binaries.

## Embedding Additional Files

### Embedding Additional Files

Sometimes an app bundle has to ship extra files that don't belong to the frontend (your `frontendDist`) or that are simply too large to embed straight into the binary. Tauri refers to such files as `resources`.

### Configuration

Pick the files you want shipped and register them through the `resources` property on the `bundle` object within your `tauri.conf.json` file.

Supplying them as a list of paths:

- Syntax
- Explanation
  tauri.conf.json

```
{  "bundle": {    "resources": [      "./path/to/some-file.txt",      "/absolute/path/to/textfile.txt",      "../relative/path/to/jsonfile.json",      "some-folder/",      "resources/**/*.md"    ]  }}
```

tauri.conf.json5

```
{  "bundle": {    "resources": [      // Will be placed to `$RESOURCE/path/to/some-file.txt`      "./path/to/some-file.txt",
      // The root in an absolute path will be replaced by `_root_`,      // so `textfile.txt` will be placed to `$RESOURCE/_root_/absolute/path/to/textfile.txt`      "/absolute/path/to/textfile.txt",
      // `..` in a relative path will be replaced by `_up_`,      // so `jsonfile.json` will be placed to `$RESOURCE/_up_/relative/path/to/textfile.txt`,      "../relative/path/to/jsonfile.json",
      // If the path is a directory, the entire directory will be copied to the `$RESOURCE` directory,      // preserving the original structures, for example:      //   - `some-folder/file.txt`                   -> `$RESOURCE/some-folder/file.txt`      //   - `some-folder/another-folder/config.json` -> `$RESOURCE/some-folder/another-folder/config.json`      // This is the same as `some-folder/**/*`      "some-folder/",
      // You can also include multiple files at once through glob patterns.      // All the `.md` files inside `resources` will be placed to `$RESOURCE/resources/`,      // preserving their original directory structures, for example:      //   - `resources/index.md`      -> `$RESOURCE/resources/index.md`      //   - `resources/docs/setup.md` -> `$RESOURCE/resources/docs/setup.md`      "resources/**/*.md"    ]  }}
```

Whatever you bundle lands under `$RESOURCES/` and keeps its original folder layout intact — so, for instance: `./path/to/some-file.txt` -> `$RESOURCE/path/to/some-file.txt`

For precise command over the destination of each file, supply a map rather than a list:

- Syntax
- Explanation
  tauri.conf.json

```
{  "bundle": {    "resources": {      "/absolute/path/to/textfile.txt": "resources/textfile.txt",      "relative/path/to/jsonfile.json": "resources/jsonfile.json",      "resources/": "",      "docs/**/*md": "website-docs/"    }  }}
```

tauri.conf.json5

```
{  "bundle": {    "resources": {      // `textfile.txt` will be placed to `$RESOURCE/resources/textfile.txt`      "/absolute/path/to/textfile.txt": "resources/textfile.txt",
      // `jsonfile.json` will be placed to `$RESOURCE/resources/jsonfile.json`      "relative/path/to/jsonfile.json": "resources/jsonfile.json",
      // Copy the entire directory to `$RESOURCE`, preserving the original structures,      // the target is "" which means it will be placed directly in the resource directory `$RESOURCE`, for example:      //   - `resources/file.txt`                -> `$RESOURCE/file.txt`      //   - `resources/some-folder/config.json` -> `$RESOURCE/some-folder/config.json`      "resources/": "",
      // When using glob patterns, the behavior is different from the list one,      // all the matching files will be placed to the target directory without preserving the original file structures      // for example:      //   - `docs/index.md`         -> `$RESOURCE/website-docs/index.md`      //   - `docs/plugins/setup.md` -> `$RESOURCE/website-docs/setup.md`      "docs/**/*md": "website-docs/"    }  }}
```

For details on how `$RESOURCE` maps to a real location on every platform, consult the reference for [`resource_dir`](https://docs.rs/tauri/latest/tauri/path/struct.PathResolver.html#method.resource_dir)

Source path syntax

Throughout the notes below, the “target resource directory” means either the string following the colon in the map form, or — when the array form is used — the rebuilt version of the source file paths.

- `"dir/file.txt"`: places the single `file.txt` file inside the target resource directory.
- `"dir/"`: pulls every file **and directories** _recursively_ into the target resource directory. Reach for this when you also want the file system layout of your files and folders kept intact.
- `"dir/*"`: grabs all files within the `dir` directory _non-recursively_ (sub-directories will be ignored) and drops them in the target resource directory.
- `"dir/**`: raises an error, since `**` matches only directories, so it can locate no files.
- `"dir/**/*"`: sweeps up every file beneath `dir` _recursively_ — covering whatever sits directly in `dir/` plus everything within each nested sub-directory — and deposits it all in the target resource directory.
- `"dir/**/**`: raises an error, since `**` matches only directories, so it can locate no files.

### Resolve resource file paths

Rather than working out a resource file's path by hand, lean on the APIs listed here to resolve it for you

- Rust
- JavaScript

Over in Rust, obtain a [`PathResolver`](https://docs.rs/tauri/latest/tauri/path/struct.PathResolver.html) instance — both [`App`](https://docs.rs/tauri/latest/tauri/struct.App.html) and [`AppHandle`](https://docs.rs/tauri/latest/tauri/struct.AppHandle.html) can hand you one — and then invoke [`PathResolver::resolve`](https://docs.rs/tauri/latest/tauri/path/struct.PathResolver.html#method.resolve):

```
tauri::Builder::default()  .setup(|app| {    let resource_path = app.path().resolve("lang/de.json", BaseDirectory::Resource)?;    Ok(())  })
```

And to do the same within a command:

```
#[tauri::command]fn hello(handle: tauri::AppHandle) {  let resource_path = handle.path().resolve("lang/de.json", BaseDirectory::Resource)?;}
```

From JavaScript, reach for [`resolveResource`](https://tauri.app/reference/javascript/api/namespacepath/#resolveresource) to work the path out:

```
import { resolveResource } from '@tauri-apps/api/path';const resourcePath = await resolveResource('lang/de.json');
```

### Path syntax

Whatever path you hand these API calls may be an ordinary relative one such as `folder/json_file.json`, which maps to `$RESOURCE/folder/json_file.json`, or something like `../relative/folder/toml_file.toml`, which maps to `$RESOURCE/_up_/relative/folder/toml_file.toml`; the rules these APIs follow are identical to the ones you'd apply writing `tauri.conf.json > bundle > resources`. For instance:

tauri.conf.json

```
{  "bundle": {    "resources": ["folder/json_file.json", "../relative/folder/toml_file.toml"]  }}
```

```
let json_path = app.path().resolve("folder/json_file.json", BaseDirectory::Resource)?;let toml_path = app.path().resolve("../relative/folder/toml_file.toml", BaseDirectory::Resource)?;
```

### Android

At present, resources live inside the APK as assets, which means what those APIs give back are not ordinary file system paths; instead Tauri prepends the dedicated URI prefix `asset://localhost/`, and that form works alongside the [`fs` plugin](https://v2.tauri.app/plugin/file-system/). With it in hand, read the files via [`FsExt::fs`](https://docs.rs/tauri-plugin-fs/latest/tauri_plugin_fs/trait.FsExt.html#tymethod.fs) in this fashion:

```
let resource_path = app.path().resolve("lang/de.json", BaseDirectory::Resource).unwrap();let json = app.fs().read_to_string(&resource_path);
```

Should you need — or simply prefer — the resource files sitting on an actual file system, hand-copy their contents out by way of the [`fs` plugin](https://v2.tauri.app/plugin/file-system/)

### Reading resource files

Here, suppose the goal is to ship some extra i18n json files arranged as follows:

```
.├── src-tauri/│   ├── tauri.conf.json│   ├── lang/│   │   ├── de.json│   │   └── en.json│   └── ...└── ...
```

tauri.conf.json

```
{  "bundle": {    "resources": ["lang/*"]  }}
```

lang/de.json

```
{  "hello": "Guten Tag!",  "bye": "Auf Wiedersehen!"}
```

### Rust

From the Rust end, you'll want a [`PathResolver`](https://docs.rs/tauri/latest/tauri/path/struct.PathResolver.html) instance, which either [`App`](https://docs.rs/tauri/latest/tauri/struct.App.html) or [`AppHandle`](https://docs.rs/tauri/latest/tauri/struct.AppHandle.html) is able to provide:

```
tauri::Builder::default()  .setup(|app| {    // The path specified must follow the same syntax as defined in    // `tauri.conf.json > bundle > resources`    let resource_path = app.path().resolve("lang/de.json", BaseDirectory::Resource)?;
    let json = std::fs::read_to_string(&resource_path).unwrap();    // Or when dealing with Android, use the file system plugin instead    // let json = app.fs().read_to_string(&resource_path);
    let lang_de: serde_json::Value = serde_json::from_str(json).unwrap();
    // This will print 'Guten Tag!' to the terminal    println!("{}", lang_de.get("hello").unwrap());
    Ok(())  })
```

```
#[tauri::command]fn hello(handle: tauri::AppHandle) -> String {    let resource_path = handle.path().resolve("lang/de.json", BaseDirectory::Resource)?;
    let json = std::fs::read_to_string(&resource_path).unwrap();    // Or when dealing with Android, use the file system plugin instead    // let json = handle.fs().read_to_string(&resource_path);
    let lang_de: serde_json::Value = serde_json::from_str(json).unwrap();
    lang_de.get("hello").unwrap()}
```

### JavaScript

Over on the JavaScript side, you have two options: either define a command resembling the one shown above and trigger it via `await invoke('hello')`, or get at the files straight through the [`fs` plugin](https://v2.tauri.app/plugin/file-system/).

Going the [`fs` plugin](https://v2.tauri.app/plugin/file-system/) route means that, beyond the [basic setup](https://v2.tauri.app/plugin/file-system/#setup), you must also set up the access control list so the plugin APIs you depend on are turned on, together with the permissions required to reach the `$RESOURCE` folder:

src-tauri/capabilities/default.json

```
{  "$schema": "../gen/schemas/desktop-schema.json",  "identifier": "main-capability",  "description": "Capability for the main window",  "windows": ["main"],  "permissions": [    "core:default",    "fs:allow-read-text-file",    "fs:allow-resource-read-recursive"  ]}
```

```
import { resolveResource } from '@tauri-apps/api/path';import { readTextFile } from '@tauri-apps/plugin-fs';
const resourcePath = await resolveResource('lang/de.json');const langDe = JSON.parse(await readTextFile(resourcePath));console.log(langDe.hello); // This will print 'Guten Tag!' to the devtools console
```

### Permissions

Because the list form swaps `../` for `_up_` in relative paths and swaps the root for `_root_` in absolute ones, the affected files end up nested in sub folders within the resource directory; to clear those paths under Tauri's [permission system](https://v2.tauri.app/security/capabilities/), apply `$RESOURCE/**/*`, which grants recursive access to them

### Examples

Given a file bundled in the following way:

tauri.conf.json

```
{  "bundle": {    "resources": ["../relative/path/to/jsonfile.json"]  }}
```

Accessing it by way of the [`fs` plugin](https://v2.tauri.app/plugin/file-system/):

src-tauri/capabilities/default.json

```
{  "$schema": "../gen/schemas/desktop-schema.json",  "identifier": "main-capability",  "description": "Capability for the main window",  "windows": ["main"],  "permissions": [    "core:default",    "fs:allow-stat",    "fs:allow-read-text-file",    "fs:allow-resource-read-recursive",    {      "identifier": "fs:scope",      "allow": ["$RESOURCE/**/*"],      "deny": ["$RESOURCE/secret.txt"]    }  ]}
```

Reaching it through the [`opener` plugin](https://v2.tauri.app/plugin/opener/) instead:

src-tauri/capabilities/default.json

```
{  "$schema": "../gen/schemas/desktop-schema.json",  "identifier": "main-capability",  "description": "Capability for the main window",  "windows": ["main"],  "permissions": [    "core:default",    {      "identifier": "opener:allow-open-path",      "allow": [        {          "path": "$RESOURCE/**/*"        }      ]    }  ]}
```

---

## Embedding External Binaries

### Embedding External Binaries

There are times when bundling external binaries makes sense — to extend what your application can do, or to spare users from having to install further dependencies themselves (Node.js or Python, say). Tauri calls such a binary a `sidecar`.

A binary here is just an executable, written in whatever programming language you like. Typical examples include Python CLI tools or API servers packaged with `pyinstaller`.

Register the binaries you want shipped by adding the `externalBin` property onto the `bundle` object inside your `tauri.conf.json`. What `externalBin` accepts is a list of strings pointing at binaries via either absolute or relative paths.

The Tauri configuration fragment below shows what a sidecar setup looks like:

src-tauri/tauri.conf.json

```
{  "bundle": {    "externalBin": [      "/absolute/path/to/sidecar",      "../relative/path/to/binary",      "binaries/my-sidecar"    ]  }}
```

For the external binary to run across every architecture you support, a same-named binary carrying a `-$TARGET_TRIPLE` suffix has to be present at the given path. As an example, `"externalBin": ["binaries/my-sidecar"]` calls for a `src-tauri/binaries/my-sidecar-x86_64-unknown-linux-gnu` executable on Linux, or `src-tauri/binaries/my-sidecar-aarch64-apple-darwin` on Mac OS running Apple Silicon.

Figure out your **current** platform's `-$TARGET_TRIPLE` suffix by issuing this command:

Terminal window

```
rustc --print host-tuple
```

That prints your host's target triple straight to the console (for example `x86_64-unknown-linux-gnu` or `aarch64-apple-darwin`).

The Node.js script below tacks the target triple onto a binary:

```
import { execSync } from 'child_process';import fs from 'fs';
const extension = process.platform === 'win32' ? '.exe' : '';
const targetTriple = execSync('rustc --print host-tuple').toString().trim();if (!targetTriple) {  console.error('Failed to determine platform target triple');}fs.renameSync(  `src-tauri/binaries/sidecar${extension}`,  `src-tauri/binaries/sidecar-${targetTriple}${extension}`);
```

Bear in mind the script breaks down once you compile for an architecture other than the one it executes on, so treat it merely as a jumping-off point for your own build scripts.

### Running it from Rust

In Rust, bring in the `tauri_plugin_shell::ShellExt` trait and invoke the `shell().sidecar()` function against the AppHandle:

```
use tauri_plugin_shell::ShellExt;use tauri_plugin_shell::process::CommandEvent;use tauri::Emitter;
let sidecar_command = app.shell().sidecar("my-sidecar").unwrap();let (mut rx, mut child) = sidecar_command  .spawn()  .expect("Failed to spawn sidecar");
tauri::async_runtime::spawn(async move {  // read events such as stdout  while let Some(event) = rx.recv().await {    if let CommandEvent::Stdout(line_bytes) = event {      let line = String::from_utf8_lossy(&line_bytes);      app        .emit("message", Some(format!("'{}'", line)))        .expect("failed to emit event");      // write to stdin      child.write("message from Rust\n".as_bytes()).unwrap();    }  }});
```

Drop this code within a Tauri command for a convenient way to pass the AppHandle along, or stash a reference to the AppHandle inside the builder script so you can get at it from anywhere else in your application.

### Running it from JavaScript

To launch the sidecar, Tauri insists that you first authorize it to invoke the `execute` or `spawn` method on the child process. Grant that authorization by opening the file `<PROJECT ROOT>/src-tauri/capabilities/default.json` and dropping the block shown below into its permissions array. Remember to label your sidecar in line with the relative path noted earlier.

src-tauri/capabilities/default.json

```
{  "permissions": [    "core:default",    {      "identifier": "shell:allow-execute",      "allow": [        {          "name": "binaries/app",          "sidecar": true        }      ]    }  ]}
```

Within the JavaScript code, pull in the `Command` class from the `@tauri-apps/plugin-shell` module and call its `sidecar` static method.

```
import { Command } from '@tauri-apps/plugin-shell';const command = Command.sidecar('binaries/my-sidecar');const output = await command.execute();
```

### Passing arguments

Handing arguments to Sidecar commands works the same way it does for a regular [Command](https://doc.rust-lang.org/std/process/struct.Command.html).

An argument comes in one of two flavors: **static** (for instance `-o` or `serve`) or **dynamic** (for instance `<file_path>` or `localhost:<PORT>`). Setting the value to `true` lets any argument through to the command, whereas `false` blocks all of them. When you supply neither `true` nor `false`, you spell out the arguments in the precise sequence you intend to call them. A static argument is written verbatim, while a dynamic one can be expressed through a regular expression.

To begin, declare in `src-tauri/capabilities/default.json` the arguments that must reach the sidecar command:

src-tauri/capabilities/default.json

```
{  "$schema": "../gen/schemas/desktop-schema.json",  "identifier": "default",  "description": "Capability for the main window",  "windows": ["main"],  "permissions": [    "core:default",    {      "identifier": "shell:allow-execute",      "allow": [        {          "args": [            "arg1",            "-a",            "--arg2",            {              "validator": "\\S+"            }          ],          "name": "binaries/my-sidecar",          "sidecar": true        }      ]    }  ]}
```

After that, invoking the sidecar command is just a matter of handing **all** the arguments over as an array.

Within Rust:

```
use tauri_plugin_shell::ShellExt;#[tauri::command]async fn call_my_sidecar(app: tauri::AppHandle) {  let sidecar_command = app    .shell()    .sidecar("my-sidecar")    .unwrap()    .args(["arg1", "-a", "--arg2", "any-string-that-matches-the-validator"]);  let (mut _rx, mut _child) = sidecar_command.spawn().unwrap();}
```

Within JavaScript:

```
import { Command } from '@tauri-apps/plugin-shell';// notice that the args array matches EXACTLY what is specified in `capabilities/default.json`.const command = Command.sidecar('binaries/my-sidecar', [  'arg1',  '-a',  '--arg2',  'any-string-that-matches-the-validator',]);const output = await command.execute();
```

---
