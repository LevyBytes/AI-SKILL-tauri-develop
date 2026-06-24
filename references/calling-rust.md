# Calling Rust (part 1)

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

### Calling Rust from the Frontend

Here you will find walkthroughs covering the ways your application frontend can talk to your Rust code. For the reverse direction — driving the frontend from within Rust — consult [Calling the Frontend from Rust](https://v2.tauri.app/develop/calling-frontend/).

For reaching Rust functions with type safety, Tauri offers a command primitive; it pairs that with a more flexible, dynamic event system.

### Commands

For invoking Rust functions out of your web app, Tauri ships a `command` system that is straightforward but capable. A command may take in arguments and hand back values; it can likewise surface errors and run as `async`.

### Basic Example

You declare commands within the `src-tauri/src/lib.rs` file. Defining one is as easy as writing a function and tagging it with `#[tauri::command]`:

src-tauri/src/lib.rs

```
#[tauri::command]fn my_custom_command() {  println!("I was invoked from JavaScript!");}
```

Next, hand the builder function a roster of every command you have, in this fashion:

src-tauri/src/lib.rs

```
#[cfg_attr(mobile, tauri::mobile_entry_point)]pub fn run() {  tauri::Builder::default()    .invoke_handler(tauri::generate_handler![my_custom_command])    .run(tauri::generate_context!())    .expect("error while running tauri application");}
```

With that done, your JavaScript code is free to call the command:

```
// When using the Tauri API npm package:import { invoke } from '@tauri-apps/api/core';
// When using the Tauri global script (if not using the npm package)// Be sure to set `app.withGlobalTauri` in `tauri.conf.json` to trueconst invoke = window.__TAURI__.core.invoke;
// Invoke the commandinvoke('my_custom_command');
```

#### Defining Commands in a Separate Module

When your app declares many components, or when those components fall into natural groups, you can keep commands in their own module rather than swelling the `lib.rs` file.

By way of illustration, let us put a command inside the `src-tauri/src/commands.rs` file:

src-tauri/src/commands.rs

```
#[tauri::command]pub fn my_custom_command() {  println!("I was invoked from JavaScript!");}
```

Over in the `lib.rs` file, declare that module and supply the command roster to match;

src-tauri/src/lib.rs

```
mod commands;
#[cfg_attr(mobile, tauri::mobile_entry_point)]pub fn run() {  tauri::Builder::default()    .invoke_handler(tauri::generate_handler![commands::my_custom_command])    .run(tauri::generate_context!())    .expect("error while running tauri application");}
```

Observe how the `commands::` prefix appears in the roster; it spells out the complete path leading to the command function.

Because the command here is named `my_custom_command`, you keep invoking it from the frontend by running `invoke("my_custom_command")` — the `commands::` prefix is dropped.

#### WASM

Should a Rust frontend call `invoke()` with no arguments, your frontend code has to be adjusted as shown below, since optional arguments are something Rust does not provide.

```
#[wasm_bindgen]extern "C" {    // invoke without arguments    #[wasm_bindgen(js_namespace = ["window", "__TAURI__", "core"], js_name = invoke)]    async fn invoke_without_args(cmd: &str) -> JsValue;
    // invoke with arguments (default)    #[wasm_bindgen(js_namespace = ["window", "__TAURI__", "core"])]    async fn invoke(cmd: &str, args: JsValue) -> JsValue;
    // They need to have different names!}
```

### Passing Arguments

Arguments are something your command handlers are able to accept:

```
#[tauri::command]fn my_custom_command(invoke_message: String) {  println!("I was invoked from JavaScript, with this message: {}", invoke_message);}
```

Pass the arguments along inside a JSON object whose keys use camelCase:

```
invoke('my_custom_command', { invokeMessage: 'Hello!' });
```

Any type is acceptable for an argument, provided it implements [`serde::Deserialize`](https://docs.serde.rs/serde/trait.Deserialize.html).

### Returning Data

Returning data is also within the reach of command handlers:

```
#[tauri::command]fn my_custom_command() -> String {  "Hello from Rust!".into()}
```

A promise comes back from the `invoke` function, and it resolves carrying whatever value was returned:

```
invoke('my_custom_command').then((message) => console.log(message));
```

Data you return may be of any type, so long as it implements [`serde::Serialize`](https://docs.serde.rs/serde/trait.Serialize.html).

#### Returning Array Buffers

Whenever the response heads back to the frontend, return values that implement [`serde::Serialize`](https://docs.serde.rs/serde/trait.Serialize.html) get serialized into JSON. Handing back something bulky — a file, say, or a downloaded HTTP response — through that path can bog your app down. For an efficient way to send array buffers instead, reach for [`tauri::ipc::Response`](https://docs.rs/tauri/2.0.0/tauri/ipc/struct.Response.html):

```
use tauri::ipc::Response;#[tauri::command]fn read_file() -> Response {  let data = std::fs::read("/path/to/file").unwrap();  tauri::ipc::Response::new(data)}
```

### Error Handling

When a handler might fail and has to be capable of reporting an error, make the function return a `Result`:

```
#[tauri::command]fn login(user: String, password: String) -> Result<String, String> {  if user == "tauri" && password == "tauri" {    // resolve    Ok("logged_in".to_string())  } else {    // reject    Err("invalid credentials".to_string())  }}
```

Should the command yield an error the promise rejects; failing that, it resolves:

```
invoke('login', { user: 'tauri', password: '0j4rijw8=' })  .then((message) => console.log(message))  .catch((error) => console.error(error));
```

Per the earlier note, anything a command returns — errors included — has to implement [`serde::Serialize`](https://docs.serde.rs/serde/trait.Serialize.html). That becomes awkward when you lean on error types from Rust's std library or from outside crates, since the bulk of error types skip that implementation. For the easy cases, lean on `map_err` to turn such errors into `String`:

```
#[tauri::command]fn my_custom_command() -> Result<(), String> {  std::fs::File::open("path/to/file").map_err(|err| err.to_string())?;  // Return `null` on success  Ok(())}
```

As that approach is hardly idiomatic, you might prefer rolling your own error type that implements `serde::Serialize`. The example that follows leans on the [`thiserror`](https://github.com/dtolnay/thiserror) crate to assemble the error type; by deriving the `thiserror::Error` trait it lets you promote enums into error types. Its documentation covers the finer points.

```
// create the error type that represents all errors possible in our program#[derive(Debug, thiserror::Error)]enum Error {  #[error(transparent)]  Io(#[from] std::io::Error)}
// we must manually implement serde::Serializeimpl serde::Serialize for Error {  fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>  where    S: serde::ser::Serializer,  {    serializer.serialize_str(self.to_string().as_ref())  }}
#[tauri::command]fn my_custom_command() -> Result<(), Error> {  // This will return an error  std::fs::File::open("path/that/does/not/exist")?;  // Return `null` on success  Ok(())}
```

One upside of a custom error type is that it lays every possible error out in the open, letting readers see at a glance which failures may occur. Down the line, that spares everyone — you included — a great deal of effort during review and refactoring.
On top of that, it hands you complete say over how your error type is serialized. The earlier snippet merely returned the error message as a string, yet you might instead tag each error with a code, making it simpler to line up with, for instance, a comparable TypeScript error enum:

```
#[derive(Debug, thiserror::Error)]enum Error {  #[error(transparent)]  Io(#[from] std::io::Error),  #[error("failed to parse as string: {0}")]  Utf8(#[from] std::str::Utf8Error),}
#[derive(serde::Serialize)]#[serde(tag = "kind", content = "message")]#[serde(rename_all = "camelCase")]enum ErrorKind {  Io(String),  Utf8(String),}
impl serde::Serialize for Error {  fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>  where    S: serde::ser::Serializer,  {    let error_message = self.to_string();    let error_kind = match self {      Self::Io(_) => ErrorKind::Io(error_message),      Self::Utf8(_) => ErrorKind::Utf8(error_message),    };    error_kind.serialize(serializer)  }}
#[tauri::command]fn read() -> Result<Vec<u8>, Error> {  let data = std::fs::read("/path/to/file")?;  Ok(data)}
```

Your frontend now receives an error object shaped as `{ kind: 'io' | 'utf8', message: string }`:

```
type ErrorKind = {  kind: 'io' | 'utf8';  message: string;};
invoke('read').catch((e: ErrorKind) => {});
```

### Async Commands

For demanding work, Tauri favors asynchronous commands, since they handle it without locking up or slowing the UI.

**If your command needs to run asynchronously, simply declare it as `async`.**

Borrowed types call for a few extra tweaks. You have two principal paths to choose from:

**Option 1**: Swap the borrowed type for a comparable non-borrowed one — say, `&str` for `String`. Certain types, such as `State<'_, Data>`, won't cooperate with this.

_Example:_

```
// Declare the async function using String instead of &str, as &str is borrowed and thus unsupported#[tauri::command]async fn my_custom_command(value: String) -> String {  // Call another async function and wait for it to finish  some_async_function().await;  value}
```

**Option 2**: Enclose the return type within a [`Result`](https://doc.rust-lang.org/std/result/index.html). Implementing this takes a touch more effort, but it covers every type.

Adopt the return type `Result<a, b>`: put in place of `a` whatever type you want back — or `()` when you mean to return `null` — and in place of `b` an error type for when things go awry, or `()` if you want no optional error coming back. As examples:

- `Result<String, ()>` to hand back a String with no error.
- `Result<(), ()>` to give back `null`.
- `Result<bool, Error>` to deliver a boolean, or an error along the lines shown in the Error Handling section earlier.

_Example:_

```
// Return a Result<String, ()> to bypass the borrowing issue#[tauri::command]async fn my_custom_command(value: &str) -> Result<String, ()> {  // Call another async function and wait for it to finish  some_async_function().await;  // Note that the return value must be wrapped in `Ok()` now.  Ok(format!(value))}
```

##### Invoking from JavaScript

Given that calling the command from JavaScript hands back a promise already, it behaves no differently from any other command:

```
invoke('my_custom_command', { value: 'Hello, Async!' }).then(() =>  console.log('Completed!'));
```

### Channels

When you need to stream data to the frontend — streamed HTTP responses, for instance — the Tauri channel is the advised tool. In the example below, a file is read and the frontend gets progress updates in 4096-byte chunks:

```
use tokio::io::AsyncReadExt;
#[tauri::command]async fn load_image(path: std::path::PathBuf, reader: tauri::ipc::Channel<&[u8]>) {  // for simplicity this example does not include error handling  let mut file = tokio::fs::File::open(path).await.unwrap();
  let mut chunk = vec![0; 4096];
  loop {    let len = file.read(&mut chunk).await.unwrap();    if len == 0 {      // Length of zero means end of file.      break;    }    reader.send(&chunk).unwrap();  }}
```

Further detail lives in the [channels documentation](https://v2.tauri.app/develop/calling-frontend/#channels).

### Accessing the WebviewWindow in Commands

A command is able to get at the `WebviewWindow` instance that triggered the message:

src-tauri/src/lib.rs

```
#[tauri::command]async fn my_custom_command(webview_window: tauri::WebviewWindow) {  println!("WebviewWindow: {}", webview_window.label());}
```

### Accessing an AppHandle in Commands

Reaching an `AppHandle` instance is also possible from a command:

src-tauri/src/lib.rs

```
#[tauri::command]async fn my_custom_command(app_handle: tauri::AppHandle) {  let app_dir = app_handle.path().app_dir();  use tauri::GlobalShortcutManager;  app_handle.global_shortcut_manager().register("CTRL + U", move || {});}
```

### Accessing Managed State

Through the `manage` function on `tauri::Builder`, Tauri takes care of state for you. A command then gets at that state by way of `tauri::State`:

src-tauri/src/lib.rs

```
struct MyState(String);
#[tauri::command]fn my_custom_command(state: tauri::State<MyState>) {  assert_eq!(state.0 == "some state value", true);}
#[cfg_attr(mobile, tauri::mobile_entry_point)]pub fn run() {  tauri::Builder::default()    .manage(MyState("some state value".into()))    .invoke_handler(tauri::generate_handler![my_custom_command])    .run(tauri::generate_context!())    .expect("error while running tauri application");}
```

### Accessing Raw Request

Tauri commands may likewise tap the entire [`tauri::ipc::Request`](https://docs.rs/tauri/2.0.0/tauri/ipc/struct.Request.html) object, which carries both the raw body payload and the request headers.

```
#[derive(Debug, thiserror::Error)]enum Error {  #[error("unexpected request body")]  RequestBodyMustBeRaw,  #[error("missing `{0}` header")]  MissingHeader(&'static str),}
impl serde::Serialize for Error {  fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>  where    S: serde::ser::Serializer,  {    serializer.serialize_str(self.to_string().as_ref())  }}
#[tauri::command]fn upload(request: tauri::ipc::Request) -> Result<(), Error> {  let tauri::ipc::InvokeBody::Raw(upload_data) = request.body() else {    return Err(Error::RequestBodyMustBeRaw);  };  let Some(authorization_header) = request.headers().get("Authorization") else {    return Err(Error::MissingHeader("Authorization"));  };
  // upload...
  Ok(())}
```

From the frontend, call invoke() with a raw request body by handing an ArrayBuffer or Uint8Array to the payload argument, and supply request headers via the third argument:

```
const data = new Uint8Array([1, 2, 3]);await __TAURI__.core.invoke('upload', data, {  headers: {    Authorization: 'apikey',  },});
```

### Creating Multiple Commands

An array of commands is what the `tauri::generate_handler!` macro expects. Registering several commands is not done by calling invoke_handler more than once — only the final such call counts. Every command has to go into one solitary call of `tauri::generate_handler!`.

src-tauri/src/lib.rs

```
#[tauri::command]fn cmd_a() -> String {  "Command a"}#[tauri::command]fn cmd_b() -> String {  "Command b"}
#[cfg_attr(mobile, tauri::mobile_entry_point)]pub fn run() {  tauri::Builder::default()    .invoke_handler(tauri::generate_handler![cmd_a, cmd_b])    .run(tauri::generate_context!())    .expect("error while running tauri application");}
```

### Complete Example

Some or every feature covered above can be brought together:

src-tauri/src/lib.rs

```
struct Database;
#[derive(serde::Serialize)]struct CustomResponse {  message: String,  other_val: usize,}
async fn some_other_function() -> Option<String> {  Some("response".into())}
#[tauri::command]async fn my_custom_command(  window: tauri::WebviewWindow,  number: usize,  database: tauri::State<'_, Database>,) -> Result<CustomResponse, String> {  println!("Called from {}", window.label());  let result: Option<String> = some_other_function().await;  if let Some(message) = result {    Ok(CustomResponse {      message,      other_val: 42 + number,    })  } else {    Err("No result".into())  }}
#[cfg_attr(mobile, tauri::mobile_entry_point)]pub fn run() {  tauri::Builder::default()    .manage(Database {})    .invoke_handler(tauri::generate_handler![my_custom_command])    .run(tauri::generate_context!())    .expect("error while running tauri application");}
```

```
import { invoke } from '@tauri-apps/api/core';
// Invocation from JavaScriptinvoke('my_custom_command', {  number: 42,})  .then((res) =>    console.log(`Message: ${res.message}, Other Val: ${res.other_val}`)  )  .catch((e) => console.error(e));
```

### Event System

For talking between your frontend and the Rust side, the event system is the more lightweight option. Events differ from commands: they are not type safe, they always run async, they return nothing, and they accept only JSON payloads.

### Global Events

Firing off a global event is a matter of reaching for either the [event.emit](https://v2.tauri.app/reference/javascript/api/namespaceevent/#emit) function or the [WebviewWindow#emit](https://v2.tauri.app/reference/javascript/api/namespacewebviewwindow/#emit) one:

```
import { emit } from '@tauri-apps/api/event';import { getCurrentWebviewWindow } from '@tauri-apps/api/webviewWindow';
// emit(eventName, payload)emit('file-selected', '/path/to/file');
const appWebview = getCurrentWebviewWindow();appWebview.emit('route-changed', { url: window.location.href });
```

### Webview Event

To send an event toward a listener that a particular webview registered, turn to the [event.emitTo](https://v2.tauri.app/reference/javascript/api/namespaceevent/#emitto) function or the [WebviewWindow#emitTo](https://v2.tauri.app/reference/javascript/api/namespacewebviewwindow/#emitto) one:

```
import { emitTo } from '@tauri-apps/api/event';import { getCurrentWebviewWindow } from '@tauri-apps/api/webviewWindow';
// emitTo(webviewLabel, eventName, payload)emitTo('settings', 'settings-update-requested', {  key: 'notification',  value: 'all',});
const appWebview = getCurrentWebviewWindow();appWebview.emitTo('editor', 'file-changed', {  path: '/path/to/file',  contents: 'file contents',});
```

### Listening to Events

APIs for subscribing to events — both global ones and those tied to a specific webview — are provided by the `@tauri-apps/api` NPM package.

-

Listening to global events

```
import { listen } from '@tauri-apps/api/event';
type DownloadStarted = {  url: string;  downloadId: number;  contentLength: number;};
listen<DownloadStarted>('download-started', (event) => {  console.log(    `downloading ${event.payload.contentLength} bytes from ${event.payload.url}`  );});
```

-

Listening to webview-specific events

```
import { getCurrentWebviewWindow } from '@tauri-apps/api/webviewWindow';
const appWebview = getCurrentWebviewWindow();appWebview.listen<string>('logged-in', (event) => {  localStorage.setItem('session-token', event.payload);});
```

For as long as the application lives, the `listen` function holds the event listener in place. When you want to quit listening for an event, invoke the `unlisten` function that the `listen` function gives back:

```
import { listen } from '@tauri-apps/api/event';
const unlisten = await listen('download-started', (event) => {});unlisten();
```

#### Common Pitfalls

##### Don’t call `unlisten()` before the listener resolves

A Promise that resolves into the `unlisten` handle is what the `listen` function returns. Calling `unlisten` synchronously, ahead of that Promise resolving, tears the handler down right away and leaves you receiving no events at all:

```
// Wrong: unlisten is called before the listener is registeredconst unlisten = listen('sync-complete', (event) => {  console.log('sync finished');});unlisten(); // unlisten is a Promise here, not a function -- the listener is not cleaned up
// Correct: await the Promise to get the unlisten handleconst unlisten = await listen('sync-complete', (event) => {  console.log('sync finished');});// Now you can store and call it later, e.g. in a cleanup functionunlisten();
```

##### Timing in setup hooks

With frameworks such as React, Vue, and Svelte, the setup or mount hook fires ahead of the component being fully rendered. Should you subscribe to events while in setup, see to it that the event handler leans on no DOM elements that are still unrendered — or push the listener registration into an effect/hook that fires once mounting is done.

```
// Wrong: DOM ref may not be available yetfunction MyComponent() {  const ref = useRef(null);  listen('scroll-to', (event) => {    ref.current.scrollIntoView(); // ref.current may be null during setup  });  return <div ref={ref} />;}
// Correct: use useEffect which runs after the component mountsfunction MyComponent() {  const ref = useRef(null);  useEffect(() => {    const unlisten = listen('scroll-to', (event) => {      ref.current?.scrollIntoView();    });    return () => {      unlisten.then((fn) => fn());    };  }, []);  return <div ref={ref} />;}
```

##### Event ordering and async listeners

Listeners fire in their registration order, yet when a listener is async and the emitter fires off several events in quick succession, those listeners can wind up handling events out of sequence. Where you need ordered, high-throughput delivery of data, weigh using [Channels](https://v2.tauri.app/develop/calling-frontend/#channels) in place of the event system.

#### Framework-specific Cleanup Examples

If a frontend framework is in play, tear down event listeners as a component unmounts so you steer clear of memory leaks and duplicated handlers.

- React
- Vue
- Svelte

```
import { useEffect, useState } from 'react';import { listen } from '@tauri-apps/api/event';
function DownloadTracker() {  const [progress, setProgress] = useState(0);
  useEffect(() => {    const unlisten = listen<number>('download-progress', (event) => {      setProgress(event.payload);    });
    return () => {      unlisten.then((fn) => fn());    };  }, []);
  return <div>Download progress: {progress}%</div>;}
```

```
<script setup lang="ts">import { ref, onMounted, onUnmounted } from 'vue';import { listen } from '@tauri-apps/api/event';
const progress = ref(0);let unlistenPromise;
onMounted(() => {  unlistenPromise = listen<number>('download-progress', (event) => {    progress.value = event.payload;  });});
onUnmounted(() => {  unlistenPromise?.then((fn) => fn());});</script>
<template>  <div>Download progress: {{ progress }}%</div></template>
```

```
<script lang="ts">import { listen } from '@tauri-apps/api/event';
let progress = $state(0);
$effect(() => {  const unlistenPromise = listen<number>(    'download-progress',    (event) => {      progress = event.payload;    }  );
  return () => {    unlistenPromise.then((unlisten) => unlisten());  };});</script>
<div>Download progress: {progress}%</div>
```

On top of that, Tauri supplies a helper function that subscribes to an event a single time only:

```
import { once } from '@tauri-apps/api/event';import { getCurrentWebviewWindow } from '@tauri-apps/api/webviewWindow';
once('ready', (event) => {});
const appWebview = getCurrentWebviewWindow();appWebview.once('ready', () => {});
```

#### Listening to Events on Rust

Listeners that you register in Rust receive both global and webview-specific events as well.

-

Listening to global events

src-tauri/src/lib.rs

```
use tauri::Listener;
#[cfg_attr(mobile, tauri::mobile_entry_point)]pub fn run() {  tauri::Builder::default()    .setup(|app| {      app.listen("download-started", |event| {        if let Ok(payload) = serde_json::from_str::<DownloadStarted>(&event.payload()) {          println!("downloading {}", payload.url);        }      });      Ok(())    })    .run(tauri::generate_context!())    .expect("error while running tauri application");}
```

-

Listening to webview-specific events

src-tauri/src/lib.rs

```
use tauri::{Listener, Manager};
#[cfg_attr(mobile, tauri::mobile_entry_point)]pub fn run() {  tauri::Builder::default()    .setup(|app| {      let webview = app.get_webview_window("main").unwrap();      webview.listen("logged-in", |event| {        let session_token = event.data;        // save token..      });      Ok(())    })    .run(tauri::generate_context!())    .expect("error while running tauri application");}
```
