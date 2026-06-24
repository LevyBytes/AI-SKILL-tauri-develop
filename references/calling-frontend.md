# Calling Frontend

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

### Calling the Frontend from Rust

The notes below walk through the ways your Rust layer can reach into the app's frontend. For the reverse direction — driving Rust from the UI — refer to [Calling Rust from the Frontend](https://v2.tauri.app/develop/calling-rust/).

From the Rust portion of a Tauri app you can reach the frontend in three ways: through Tauri's event system, by way of channels, or by running JavaScript directly.

### Event System

Bundled with Tauri is a lightweight event system that lets Rust and the frontend talk to each other in both directions.

This mechanism suits cases where only modest amounts of data get streamed, or where you want a many-to-many producer/consumer arrangement — a push notification feed, for instance.

If you need very low latency or heavy throughput, the event system is the wrong tool; turn instead to the channels section, whose design is tuned for streaming.

A Tauri event differs from a Tauri command in several respects: events carry no strong typing, their payloads are always JSON strings (which makes them ill-suited to larger messages), and the [capabilities](https://v2.tauri.app/security/capabilities/) system — which would let you finely govern event data and channels — does not apply to them.

Both the [AppHandle](https://docs.rs/tauri/2.0.0/tauri/struct.AppHandle.html) and [WebviewWindow](https://docs.rs/tauri/2.0.0/tauri/webview/struct.WebviewWindow.html) types provide the event-system traits [Listener](https://docs.rs/tauri/2.0.0/tauri/trait.Listener.html) and [Emitter](https://docs.rs/tauri/2.0.0/tauri/trait.Emitter.html).

An event comes in two flavors: global ones reach every listener, while webview-specific ones go only to the webview whose label matches.

### Global Events

Firing a global event is done through the [Emitter#emit](https://docs.rs/tauri/2.0.0/tauri/trait.Emitter.html#tymethod.emit) function:

src-tauri/src/lib.rs

```
use tauri::{AppHandle, Emitter};
#[tauri::command]fn download(app: AppHandle, url: String) {  app.emit("download-started", &url).unwrap();  for progress in [1, 15, 50, 80, 100] {    app.emit("download-progress", progress).unwrap();  }  app.emit("download-finished", &url).unwrap();}
```

### Webview Event

When you want to reach a listener that one particular webview registered, reach for the [Emitter#emit_to](https://docs.rs/tauri/2.0.0/tauri/trait.Emitter.html#tymethod.emit_to) function:

src-tauri/src/lib.rs

```
use tauri::{AppHandle, Emitter};
#[tauri::command]fn login(app: AppHandle, user: String, password: String) {  let authenticated = user == "tauri-apps" && password == "tauri";  let result = if authenticated { "loggedIn" } else { "invalidCredentials" };  app.emit_to("login", "login-result", result).unwrap();}
```

You may also dispatch an event to a set of webviews via [Emitter#emit_filter](https://docs.rs/tauri/2.0.0/tauri/trait.Emitter.html#tymethod.emit_filter). The snippet below sends an open-file event to the main and file-viewer webviews:

src-tauri/src/lib.rs

```
use tauri::{AppHandle, Emitter, EventTarget};
#[tauri::command]fn open_file(app: AppHandle, path: std::path::PathBuf) {  app.emit_filter("open-file", path, |target| match target {    EventTarget::WebviewWindow { label } => label == "main" || label == "file-viewer",    _ => false,  }).unwrap();}
```

### Event Payload

Any [serializable](https://serde.rs/impl-serialize.html) type that additionally implements [Clone](https://doc.rust-lang.org/std/clone/trait.Clone.html) is acceptable as a payload. To carry richer data on every event, let's extend the earlier download sample so it ships a structured object:

src-tauri/src/lib.rs

```
use tauri::{AppHandle, Emitter};use serde::Serialize;
#[derive(Clone, Serialize)]#[serde(rename_all = "camelCase")]struct DownloadStarted<'a> {  url: &'a str,  download_id: usize,  content_length: usize,}
#[derive(Clone, Serialize)]#[serde(rename_all = "camelCase")]struct DownloadProgress {  download_id: usize,  chunk_length: usize,}
#[derive(Clone, Serialize)]#[serde(rename_all = "camelCase")]struct DownloadFinished {  download_id: usize,}
#[tauri::command]fn download(app: AppHandle, url: String) {  let content_length = 1000;  let download_id = 1;
  app.emit("download-started", DownloadStarted {    url: &url,    download_id,    content_length  }).unwrap();
  for chunk_length in [15, 150, 35, 500, 300] {    app.emit("download-progress", DownloadProgress {      download_id,      chunk_length,    }).unwrap();  }
  app.emit("download-finished", DownloadFinished { download_id }).unwrap();}
```

### Listening to Events

Listening for events is supported by Tauri on both the webview side and the Rust side.

#### Listening to Events on the Frontend

For subscribing to global as well as webview-scoped events, the `@tauri-apps/api` NPM package supplies the relevant APIs.

-

Subscribing to global events

```
import { listen } from '@tauri-apps/api/event';
type DownloadStarted = {  url: string;  downloadId: number;  contentLength: number;};
listen<DownloadStarted>('download-started', (event) => {  console.log(    `downloading ${event.payload.contentLength} bytes from ${event.payload.url}`  );});
```

-

Subscribing to webview-scoped events

```
import { getCurrentWebviewWindow } from '@tauri-apps/api/webviewWindow';
const appWebview = getCurrentWebviewWindow();appWebview.listen<string>('logged-in', (event) => {  localStorage.setItem('session-token', event.payload);});
```

A handler set up with `listen` stays subscribed for as long as the app is alive. When you want to drop that subscription, invoke the `unlisten` function that `listen` hands back:

```
import { listen } from '@tauri-apps/api/event';
const unlisten = await listen('download-started', (event) => {});unlisten();
```

#### Common Pitfalls

##### Avoid invoking `unlisten()` until the listener has resolved

What `listen` gives back is a Promise that eventually yields the `unlisten` handle. Should you call `unlisten` synchronously while that Promise is still pending, the handler gets torn down right away and no events ever reach you:

```
// Wrong: unlisten is called before the listener is registeredconst unlisten = listen('sync-complete', (event) => {  console.log('sync finished');});unlisten(); // unlisten is a Promise here, not a function -- the listener is not cleaned up
// Correct: await the Promise to get the unlisten handleconst unlisten = await listen('sync-complete', (event) => {  console.log('sync finished');});// Now you can store and call it later, e.g. in a cleanup functionunlisten();
```

##### Timing in setup hooks

Across frameworks such as React, Vue, and Svelte, the mount or setup hook fires ahead of the component being completely rendered. So if you register listeners inside setup, either keep the handler from touching DOM nodes that aren't on the page yet, or push the registration into an effect or hook that fires once mounting has finished.

```
// Wrong: DOM ref may not be available yetfunction MyComponent() {  const ref = useRef(null);  listen('scroll-to', (event) => {    ref.current.scrollIntoView(); // ref.current may be null during setup  });  return <div ref={ref} />;}
// Correct: use useEffect which runs after the component mountsfunction MyComponent() {  const ref = useRef(null);  useEffect(() => {    const unlisten = listen('scroll-to', (event) => {      ref.current?.scrollIntoView();    });    return () => {      unlisten.then((fn) => fn());    };  }, []);  return <div ref={ref} />;}
```

##### Event ordering and async listeners

Registration order determines the order in which listeners fire, yet when a listener is async and the emitter pushes several events back-to-back, those events can end up handled out of sequence. Where you need ordered delivery at high throughput, lean on [Channels](https://v2.tauri.app/develop/calling-frontend/#channels) rather than the event system.

#### Framework-specific Cleanup Examples

With any frontend framework, tearing down event listeners as a component unmounts is important so you don't leak memory or end up with duplicate handlers.

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

Tauri also ships a helper that subscribes to an event just a single time:

```
import { once } from '@tauri-apps/api/event';import { getCurrentWebviewWindow } from '@tauri-apps/api/webviewWindow';
once('ready', (event) => {});
const appWebview = getCurrentWebviewWindow();appWebview.once('ready', () => {});
```

#### Listening to Events on Rust

Listeners that you register on the Rust side receive global and webview-specific events too.

-

Subscribing to global events

src-tauri/src/lib.rs

```
use tauri::Listener;
#[cfg_attr(mobile, tauri::mobile_entry_point)]pub fn run() {  tauri::Builder::default()    .setup(|app| {      app.listen("download-started", |event| {        if let Ok(payload) = serde_json::from_str::<DownloadStarted>(&event.payload()) {          println!("downloading {}", payload.url);        }      });      Ok(())    })    .run(tauri::generate_context!())    .expect("error while running tauri application");}
```

-

Subscribing to webview-scoped events

src-tauri/src/lib.rs

```
use tauri::{Listener, Manager};
#[cfg_attr(mobile, tauri::mobile_entry_point)]pub fn run() {  tauri::Builder::default()    .setup(|app| {      let webview = app.get_webview_window("main").unwrap();      webview.listen("logged-in", |event| {        let session_token = event.data;        // save token..      });      Ok(())    })    .run(tauri::generate_context!())    .expect("error while running tauri application");}
```

A subscription made with `listen` persists for the whole run of the application. To end it, reach for the `unlisten` function:

```
// unlisten outside of the event handler scope:let event_id = app.listen("download-started", |event| {});app.unlisten(event_id);
// unlisten when some event criteria is matchedlet handle = app.handle().clone();app.listen("status-changed", |event| {  if event.data == "ready" {    handle.unlisten(event.id);  }});
```

There is likewise a convenience helper on this side for catching an event one solitary time:

```
app.once("ready", |event| {  println!("app is ready");});
```

Here the listener gets dropped the moment it fires for the first time.

### Channels

As a basic, app-wide two-way channel of communication, the event system is built for simplicity. Because it works by running JavaScript directly behind the scenes, it can fall short when you need to ship a lot of data.

Channels, by contrast, are built for speed and guarantee ordered delivery. Internally they back streaming work like download progress, child-process output, and WebSocket traffic.

Let's recast the download command from earlier so it relies on channels in place of the event system:

src-tauri/src/lib.rs

```
use tauri::{AppHandle, ipc::Channel};use serde::Serialize;
#[derive(Clone, Serialize)]#[serde(rename_all = "camelCase", rename_all_fields = "camelCase", tag = "event", content = "data")]enum DownloadEvent<'a> {  Started {    url: &'a str,    download_id: usize,    content_length: usize,  },  Progress {    download_id: usize,    chunk_length: usize,  },  Finished {    download_id: usize,  },}
#[tauri::command]fn download(app: AppHandle, url: String, on_event: Channel<DownloadEvent>) {  let content_length = 1000;  let download_id = 1;
  on_event.send(DownloadEvent::Started {    url: &url,    download_id,    content_length,  }).unwrap();
  for chunk_length in [15, 150, 35, 500, 300] {    on_event.send(DownloadEvent::Progress {      download_id,      chunk_length,    }).unwrap();  }
  on_event.send(DownloadEvent::Finished { download_id }).unwrap();}
```

As you invoke the download command, you have to construct the channel and pass it along as an argument:

```
import { invoke, Channel } from '@tauri-apps/api/core';
type DownloadEvent =  | {      event: 'started';      data: {        url: string;        downloadId: number;        contentLength: number;      };    }  | {      event: 'progress';      data: {        downloadId: number;        chunkLength: number;      };    }  | {      event: 'finished';      data: {        downloadId: number;      };    };
const onEvent = new Channel<DownloadEvent>();onEvent.onmessage = (message) => {  console.log(`got download event ${message.event}`);};
await invoke('download', {  url: 'https://raw.githubusercontent.com/tauri-apps/tauri/dev/crates/tauri-schema-generator/schemas/config.schema.json',  onEvent,});
```

### Evaluating JavaScript

For running arbitrary JavaScript straight inside the webview context, the [`WebviewWindow#eval`](https://docs.rs/tauri/2.0.0/tauri/webview/struct.WebviewWindow.html#method.eval) function is what you want:

src-tauri/src/lib.rs

```
use tauri::Manager;
tauri::Builder::default()  .setup(|app| {    let webview = app.get_webview_window("main").unwrap();    webview.eval("console.log('hello from Rust')")?;    Ok(())  })
```

When the script you're evaluating grows more involved and has to draw on data from Rust objects, the [serialize-to-javascript](https://docs.rs/serialize-to-javascript/latest/serialize_to_javascript/) crate is the recommended choice.

---
