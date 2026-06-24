# Plugins

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

### Plugin Development

Through a plugin you can tap into the Tauri lifecycle, surface Rust functionality that leans on the web view APIs, service commands written in Rust, Kotlin, or Swift, and a great deal beyond that.

What Tauri provides is a windowing layer with web view support, a channel for passing messages back and forth between the Rust process and the web view, and an event system paired with a handful of utilities meant to make building easier. The Tauri core is deliberately kept lean, leaving out capabilities that aren't universally required. In their place it supplies a way to bolt extra functionality onto a Tauri application, and that mechanism is the plugin.

Each Tauri plugin pairs a Cargo crate with an optional NPM package supplying API bindings for the plugin's commands and events. A plugin project may also bundle an Android library project together with a Swift package targeting iOS. For details on building plugins that target Android and iOS, see the [Mobile Plugin Development guide](https://v2.tauri.app/develop/plugins/develop-mobile/).

### Naming Convention

A Tauri plugin's name carries a prefix in front of it. You declare that plugin name within the plugin configuration at [`tauri.conf.json > plugins`](https://v2.tauri.app/reference/config/#pluginconfig).

Out of the box, Tauri puts the `tauri-plugin-` prefix on your plugin crate. Doing so makes the plugin easier for the Tauri community to find and lets the Tauri CLI work with it. As you set up a fresh plugin project, you have to supply its name. The crate that gets generated is named `tauri-plugin-{plugin-name}`, while the JavaScript NPM package becomes `tauri-plugin-{plugin-name}-api` (though where you can, we suggest reaching for an [NPM scope](https://docs.npmjs.com/about-scopes)). For NPM packages, Tauri's convention names them `@scope-name/plugin-{plugin-name}`.

### Initialize Plugin Project

Scaffold a fresh plugin project by running `plugin new`. Should you have no need for the NPM package, pass the `--no-api` CLI flag. To set the plugin up with Android support, iOS support, or both, reach for the `--android` and/or `--ios` flags.

Once that's installed, the command below creates a plugin project:

- npm

```
npx @tauri-apps/cli plugin new [name]
```

That sets the plugin up inside the `tauri-plugin-[name]` directory, and according to whichever CLI flags you passed, the project that results takes this shape:

```
. tauri-plugin-[name]/├── src/                - Rust code│ ├── commands.rs       - Defines the commands the webview can use| ├── desktop.rs        - Desktop implementation| ├── error.rs          - Default error type to use in returned results│ ├── lib.rs            - Re-exports appropriate implementation, setup state...│ ├── mobile.rs         - Mobile implementation│ └── models.rs         - Shared structs├── permissions/        - This will host (generated) permission files for commands├── android             - Android library├── ios                 - Swift package├── guest-js            - Source code of the JavaScript API bindings├── dist-js             - Transpiled assets from guest-js├── Cargo.toml          - Cargo crate metadata└── package.json        - NPM package metadata
```

When a plugin already exists and you want to give it Android or iOS capabilities, the `plugin android add` and `plugin ios add` commands will scaffold the mobile library projects for you and walk you through the edits that are required.

### Mobile Plugin Development

Native mobile code authored in Kotlin (or Java) and in Swift can be executed by plugins. By default, the plugin template ships with both a Kotlin-based Android library project and a Swift package. Bundled in is a sample mobile command that demonstrates how Rust code can kick off its execution.

For a deeper look at building plugins aimed at mobile, consult the [Mobile Plugin Development guide](https://v2.tauri.app/develop/plugins/develop-mobile/).

### Plugin Configuration

Within whichever Tauri application makes use of the plugin, you declare the plugin configuration inside `tauri.conf.json`, where `plugin-name` stands for the plugin's name:

```
{  "build": { ... },  "tauri": { ... },  "plugins": {    "plugin-name": {      "timeout": 30    }  }}
```

You attach the plugin's configuration to the `Builder`, and it gets parsed while the app runs. Below is a sample showing the `Config` struct put to work to describe the plugin configuration:

src/lib.rs

```
use serde::Deserialize;use tauri::{    plugin::{Builder, TauriPlugin},    Runtime,};
// Define the plugin config#[derive(Deserialize)]pub struct Config {  timeout: usize,}
pub fn init<R: Runtime>() -> TauriPlugin<R, Config> {  // Make the plugin config optional  // by using `Builder::<R, Option<Config>>` instead  Builder::<R, Config>::new("<plugin-name>")    .setup(|app, api| {      let timeout = api.config().timeout;      Ok(())    })    .build()}
```

### Lifecycle Events

A plugin is able to attach itself to a number of lifecycle events:

- setup: the plugin is undergoing initialization
- on_navigation: the web view is trying to carry out navigation
- on_webview_ready: a new window is in the process of being created
- on_event: events from the event loop
- on_drop: the plugin is being torn down

Mobile plugins have further [lifecycle events for mobile plugins](https://v2.tauri.app/develop/plugins/develop-mobile/#lifecycle-events).

### setup

- **When**: the plugin is undergoing initialization
- **Why**: hook up mobile plugins, handle state, kick off background work
  src/lib.rs

```
use tauri::{Manager, plugin::Builder};use std::{collections::HashMap, sync::Mutex, time::Duration};
struct DummyStore(Mutex<HashMap<String, String>>);
Builder::new("<plugin-name>")  .setup(|app, api| {    app.manage(DummyStore(Default::default()));
    let app_ = app.clone();    std::thread::spawn(move || {      loop {        app_.emit("tick", ());        std::thread::sleep(Duration::from_secs(1));      }    });
    Ok(())  })
```

### on_navigation

- **When**: the web view is trying to carry out navigation
- **Why**: check that the navigation is valid or keep tabs on URL changes

When you return `false`, the navigation is canceled.

src/lib.rs

```
use tauri::plugin::Builder;
Builder::new("<plugin-name>")  .on_navigation(|window, url| {    println!("window {} is navigating to {}", window.label(), url);    // Cancels the navigation if forbidden    url.scheme() != "forbidden"  })
```

### on_webview_ready

- **When**: a new window has finished being created
- **Why**: run a startup script against each window
  src/lib.rs

```
use tauri::plugin::Builder;
Builder::new("<plugin-name>")  .on_webview_ready(|window| {    window.listen("content-loaded", |event| {      println!("webview content has been loaded");    });  })
```

### on_event

- **When**: events from the event loop
- **Why**: respond to core events like window events, menu events, and a requested application exit

This lifecycle hook lets you receive notification of every event loop [events](https://docs.rs/tauri/2.0.0/tauri/enum.RunEvent.html).

src/lib.rs

```
use std::{collections::HashMap, fs::write, sync::Mutex};use tauri::{plugin::Builder, Manager, RunEvent};
struct DummyStore(Mutex<HashMap<String, String>>);
Builder::new("<plugin-name>")  .setup(|app, _api| {    app.manage(DummyStore(Default::default()));    Ok(())  })  .on_event(|app, event| {    match event {      RunEvent::ExitRequested { api, .. } => {        // user requested a window to be closed and there's no windows left
        // we can prevent the app from exiting:        api.prevent_exit();      }      RunEvent::Exit => {        // app is going to exit, you can cleanup here
        let store = app.state::<DummyStore>();        write(          app.path().app_local_data_dir().unwrap().join("store.json"),          serde_json::to_string(&*store.0.lock().unwrap()).unwrap(),        )        .unwrap();      }      _ => {}    }  })
```

### on_drop

- **When**: the plugin is being torn down
- **Why**: run code once the plugin has been destroyed

For further detail, refer to [`Drop`](https://doc.rust-lang.org/std/ops/trait.Drop.html).

src/lib.rs

```
use tauri::plugin::Builder;
Builder::new("<plugin-name>")  .on_drop(|app| {    // plugin has been destroyed...  })
```

### Exposing Rust APIs

Whatever plugin APIs you declare in the project's `desktop.rs` and `mobile.rs` get handed to the user as a struct bearing the plugin's name in pascal case. At plugin setup time, one instance of that struct is built and held as state, which means users can pull it out whenever they like via a `Manager` instance (for example `AppHandle`, `App`, or` Window`) by way of the extension trait the plugin declares.

To illustrate, the [`global-shortcut plugin`](https://v2.tauri.app/plugin/global-shortcut/) declares a `GlobalShortcut` struct, and you reach it through the `global_shortcut` method on the `GlobalShortcutExt` trait:

src-tauri/src/lib.rs

```
use tauri_plugin_global_shortcut::GlobalShortcutExt;
tauri::Builder::default()  .plugin(tauri_plugin_global_shortcut::init())  .setup(|app| {    app.global_shortcut().register(...);    Ok(())  })
```

### Adding Commands

You declare commands in the `commands.rs` file, and they behave just like ordinary Tauri application commands. They can reach the AppHandle and Window instances straight away, get at state, and accept input in the very same fashion as application commands do. For more on Tauri commands, consult the [Commands guide](https://v2.tauri.app/develop/calling-rust/).

The following command demonstrates obtaining the `AppHandle` and `Window` instance through dependency injection, and it accepts a pair of input parameters (`on_progress` and `url`):

src/commands.rs

```
use tauri::{command, ipc::Channel, AppHandle, Runtime, Window};
#[command]async fn upload<R: Runtime>(app: AppHandle<R>, window: Window<R>, on_progress: Channel, url: String) {  // implement command logic here  on_progress.send(100).unwrap();}
```

So that the webview can reach the command, you have to wire it into the `invoke_handler()` call inside `lib.rs`:

src/lib.rs

```
Builder::new("<plugin-name>")    .invoke_handler(tauri::generate_handler![commands::upload])
```

Declare a binding function within `webview-src/index.ts` so the people using your plugin can invoke the command from JavaScript without trouble:

```
import { invoke, Channel } from '@tauri-apps/api/core'
export async function upload(url: string, onProgressHandler: (progress: number) => void): Promise<void> {  const onProgress = new Channel<number>()  onProgress.onmessage = onProgressHandler  await invoke('plugin:<plugin-name>|upload', { url, onProgress })}
```

Make a point of compiling the TypeScript code before you try testing it.

### Command Permissions

Out of the box, the frontend cannot reach your commands. Attempt to run any of them and you'll be met with a denied error rejection. For commands to genuinely be exposed, you additionally have to declare permissions that grant access to each one.

#### Permission Files

You express permissions as JSON or TOML files placed within the `permissions` directory. Any given file may declare a set of permissions, a set of permission sets, and the default permission for your plugin.

##### Permissions

What a permission spells out is the privileges held by your plugin commands. It is able to grant or refuse a set of commands and tie together scopes that are global as well as ones tied to particular commands.

permissions/start-server.toml

```
"$schema" = "schemas/schema.json"
[[permission]]identifier = "allow-start-server"description = "Enables the start_server command."commands.allow = ["start_server"]
[[permission]]identifier = "deny-start-server"description = "Denies the start_server command."commands.deny = ["start_server"]
```

##### Scope

By means of scopes, your plugin can place finer-grained limits on individual commands. Within each permission you may declare a set of scope objects, every one of which marks something as granted or refused, whether for a single command or across the whole plugin.

Below, let's lay out a sample struct meant to carry scope data covering the set of binaries that a `shell` plugin may spawn:

src/scope.rs

```
#[derive(Debug, schemars::JsonSchema)]pub struct Entry {    pub binary: String,}
```

###### Command Scope

Anyone consuming your plugin is able to set a scope for one particular command within their capability file (refer to the [documentation](https://v2.tauri.app/reference/acl/scope/)). To read back that command-specific scope, use the [`tauri::ipc::CommandScope`](https://docs.rs/tauri/2.0.0/tauri/ipc/struct.CommandScope.html) struct:

src/commands.rs

```
use tauri::ipc::CommandScope;use crate::scope::Entry;
async fn spawn<R: tauri::Runtime>(app: tauri::AppHandle<R>, command_scope: CommandScope<'_, Entry>) -> Result<()> {  let allowed = command_scope.allows();  let denied = command_scope.denies();  todo!()}
```

###### Global Scope

Should a permission name no commands at all to grant or refuse, it counts as a scope permission, and its only job is to lay out a global scope for your plugin:

permissions/spawn-node.toml

```
[[permission]]identifier = "allow-spawn-node"description = "This scope permits spawning the `node` binary."
[[permission.scope.allow]]binary = "node"
```

Reading the global scope is done by way of the [`tauri::ipc::GlobalScope`](https://docs.rs/tauri/2.0.0/tauri/ipc/struct.GlobalScope.html) struct:

src/commands.rs

```
use tauri::ipc::GlobalScope;use crate::scope::Entry;
async fn spawn<R: tauri::Runtime>(app: tauri::AppHandle<R>, scope: GlobalScope<'_, Entry>) -> Result<()> {  let allowed = scope.allows();  let denied = scope.denies();  todo!()}
```

###### Schema

For the scope entry to produce a JSON schema, it leans on the `schemars` dependency, which lets those consuming the plugin understand the scope's format and get autocomplete inside their IDEs.

Setting up the schema starts with adding the dependency to your Cargo.toml file:

```
# we need to add schemars to both dependencies and build-dependencies because the scope.rs module is shared between the app code and build script[dependencies]schemars = "0.8"
[build-dependencies]schemars = "0.8"
```

Inside your build script, drop in the code below:

build.rs

```
#[path = "src/scope.rs"]mod scope;
const COMMANDS: &[&str] = &[];
fn main() {    tauri_plugin::Builder::new(COMMANDS)        .global_scope_schema(schemars::schema_for!(scope::Entry))        .build();}
```

##### Permission Sets

A permission set bundles individual permissions together, giving users a more abstract handle for managing your plugin. As an example, when one API draws on several commands, or when a group of commands hang together logically, it's worth declaring a set that gathers them:

permissions/websocket.toml

```
"$schema" = "schemas/schema.json"[[set]]identifier = "allow-websocket"description = "Allows connecting and sending messages through a WebSocket"permissions = ["allow-connect", "allow-send"]
```

##### Default Permission

A special permission set carrying the identifier `default` serves as the default permission. The advice is to switch on the commands that are required right out of the box. As an illustration, without the `request` command granted, the `http` plugin serves no purpose:

permissions/default.toml

```
"$schema" = "schemas/schema.json"[default]description = "Allows making HTTP requests"permissions = ["allow-request"]
```

#### Autogenerated Permissions

Reaching for the autogeneration option laid out in your plugin's build script within the `build.rs` file is the simplest route to declaring permissions for each of your commands. Within the `COMMANDS` const, list out your commands in snake_case (these ought to line up with the command function name), and Tauri takes care of producing an `allow-$commandname` plus a `deny-$commandname` permissions on its own.

In the example that follows, the `allow-upload` and `deny-upload` permissions are produced:

src/commands.rs

```
const COMMANDS: &[&str] = &["upload"];
fn main() {    tauri_plugin::Builder::new(COMMANDS).build();}
```

For more on this, refer to the [Permissions Overview](https://v2.tauri.app/security/permissions/) documentation.

### Managing State

Handling state inside a plugin works exactly as it does in a Tauri application. For more on this, see the [State Management guide](https://v2.tauri.app/develop/state-management/).

---
