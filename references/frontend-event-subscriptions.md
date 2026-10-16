# Frontend event listener registration

[Topic context](rust-frontend-communication-index.md) / [Previous source section](rust-event-emission.md) / [Next source section](rust-to-frontend-listener-lifecycle.md)

Context: Tauri v2: Frontend event listener registration. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-listening-to-events"></a> <a id="source-listening-to-events-on-the-frontend"></a>

### Listening to Events

Listening for events is supported by Tauri on both the webview side and the Rust side.

#### Listening to Events on the Frontend

For subscribing to global as well as webview-scoped events, the `@tauri-apps/api` NPM package supplies the relevant APIs.

-

Subscribing to global events

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

Subscribing to webview-scoped events

```ts
import { getCurrentWebviewWindow } from '@tauri-apps/api/webviewWindow';

const appWebview = getCurrentWebviewWindow();
appWebview.listen<string>('logged-in', (event) => {
  localStorage.setItem('session-token', event.payload);
});
```

A handler set up with `listen` stays subscribed for as long as the app is alive. When you want to drop that subscription, invoke the `unlisten` function that `listen` hands back:

```js
import { listen } from '@tauri-apps/api/event';

const unlisten = await listen('download-started', (event) => {});
unlisten();
```

