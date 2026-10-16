# Frontend event emission and listener registration

[Rust commands and events](rust-commands-events-index.md) / [Previous source section](rust-commands-requests-registration.md) / [Next source section](frontend-to-rust-listener-lifecycle.md)

Context: Tauri v2: Frontend event emission and listener registration. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-event-system"></a> <a id="source-global-events"></a> <a id="source-webview-event"></a> <a id="source-listening-to-events"></a>

### Event System

For talking between your frontend and the Rust side, the event system is the more lightweight option. Events differ from commands: they are not type safe, they always run async, they return nothing, and they accept only JSON payloads.

### Global Events

Firing off a global event is a matter of reaching for either the [event.emit](https://v2.tauri.app/reference/javascript/api/namespaceevent/#emit) function or the [WebviewWindow#emit](https://v2.tauri.app/reference/javascript/api/namespacewebviewwindow/#emit) one:

```js
import { emit } from '@tauri-apps/api/event';
import { getCurrentWebviewWindow } from '@tauri-apps/api/webviewWindow';

// emit(eventName, payload)
emit('file-selected', '/path/to/file');

const appWebview = getCurrentWebviewWindow();
appWebview.emit('route-changed', { url: window.location.href });
```

### Webview Event

To send an event toward a listener that a particular webview registered, turn to the [event.emitTo](https://v2.tauri.app/reference/javascript/api/namespaceevent/#emitto) function or the [WebviewWindow#emitTo](https://v2.tauri.app/reference/javascript/api/namespacewebviewwindow/#emitto) one:

```js
import { emitTo } from '@tauri-apps/api/event';
import { getCurrentWebviewWindow } from '@tauri-apps/api/webviewWindow';

// emitTo(webviewLabel, eventName, payload)
emitTo('settings', 'settings-update-requested', {
  key: 'notification',
  value: 'all',
});

const appWebview = getCurrentWebviewWindow();
appWebview.emitTo('editor', 'file-changed', {
  path: '/path/to/file',
  contents: 'file contents',
});
```

### Listening to Events

APIs for subscribing to events — both global ones and those tied to a specific webview — are provided by the `@tauri-apps/api` NPM package.

-

Listening to global events

```ts
import { listen } from '@tauri-apps/api/event';

type DownloadStarted = {
  url: string;
  downloadId: number;
  contentLength: number;
};

listen<DownloadStarted>('download-started', (event) => {
  console.log(
    `downloading ${event.payload.contentLength} bytes from ${event.payload.url}`
  );
});
```

-

Listening to webview-specific events

```ts
import { getCurrentWebviewWindow } from '@tauri-apps/api/webviewWindow';

const appWebview = getCurrentWebviewWindow();
appWebview.listen<string>('logged-in', (event) => {
  localStorage.setItem('session-token', event.payload);
});
```

For as long as the application lives, the `listen` function holds the event listener in place. When you want to quit listening for an event, invoke the `unlisten` function that the `listen` function gives back:

```js
import { listen } from '@tauri-apps/api/event';

const unlisten = await listen('download-started', (event) => {});
unlisten();
```

