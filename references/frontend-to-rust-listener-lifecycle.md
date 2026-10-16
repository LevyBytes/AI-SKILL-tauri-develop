# Listener timing, ordering and framework cleanup

[Rust commands and events](rust-commands-events-index.md) / [Previous source section](frontend-event-emission-subscriptions.md) / [Next source section](rust-event-listener-registration.md)

Context: Tauri v2: Listener timing, ordering and framework cleanup. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-common-pitfalls"></a> <a id="source-dont-call-unlisten-before-the-listener-resolves"></a> <a id="source-timing-in-setup-hooks"></a> <a id="source-event-ordering-and-async-listeners"></a> <a id="source-framework-specific-cleanup-examples"></a>

#### Common Pitfalls

##### Don’t call `unlisten()` before the listener resolves

`listen()` returns a Promise, not an unlisten function. Calling that Promise raises a TypeError and does not clean up the listener. Await registration, then retain the returned function for cleanup.

```js
import { listen } from '@tauri-apps/api/event';

const unlisten = await listen('sync-complete', () => {
  console.log('sync finished');
});
// Keep this handle until the subscription is no longer needed.
unlisten();
```

##### Timing in setup hooks

Register DOM-dependent listeners after mounting, such as in React useEffect, Vue onMounted or a Svelte effect. React render and Vue setup can run before DOM nodes exist. Guard DOM references and release the subscription when the component unmounts.

```tsx
import { useEffect, useRef } from 'react';
import { listen } from '@tauri-apps/api/event';

function MyComponent() {
  const ref = useRef<HTMLDivElement>(null);
  useEffect(() => {
    const pending = listen('scroll-to', () => {
      ref.current?.scrollIntoView();
    });
    return () => { pending.then((unlisten) => unlisten()); };
  }, []);
  return <div ref={ref} />;
}
```

##### Event ordering and async listeners

Listeners fire in their registration order, yet when a listener is async and the emitter fires off several events in quick succession, those listeners can wind up handling events out of sequence. Where you need ordered, high-throughput delivery of data, weigh using [Channels](https://v2.tauri.app/develop/calling-frontend/#channels) in place of the event system.

#### Framework-specific Cleanup Examples

If a frontend framework is in play, tear down event listeners as a component unmounts so you steer clear of memory leaks and duplicated handlers.

- React
- Vue

- [React event listener cleanup](react-event-listener-cleanup.md): Open when registering and cleaning up a listener in React.
- [Vue event listener cleanup](vue-event-listener-cleanup.md): Open when registering and cleaning up a listener in Vue.
- [Svelte event listener cleanup](svelte-event-listener-cleanup.md): Open when registering and cleaning up a listener in Svelte.

On top of that, Tauri supplies a helper function that subscribes to an event a single time only:

```js
import { once } from '@tauri-apps/api/event';
import { getCurrentWebviewWindow } from '@tauri-apps/api/webviewWindow';

once('ready', (event) => {});

const appWebview = getCurrentWebviewWindow();
appWebview.once('ready', () => {});
```

