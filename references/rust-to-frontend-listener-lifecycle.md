# Frontend listener timing, ordering and framework cleanup

[Topic context](rust-frontend-communication-index.md) / [Previous source section](frontend-event-subscriptions.md) / [Next source section](rust-event-subscription-lifecycle.md)

Context: Tauri v2: Frontend listener timing, ordering and framework cleanup. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-common-pitfalls"></a> <a id="source-avoid-invoking-unlisten-until-the-listener-has-resolved"></a> <a id="source-timing-in-setup-hooks"></a> <a id="source-event-ordering-and-async-listeners"></a> <a id="source-framework-specific-cleanup-examples"></a>

#### Common Pitfalls

##### Avoid invoking `unlisten()` until the listener has resolved

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

Registration order determines the order in which listeners fire, yet when a listener is async and the emitter pushes several events back-to-back, those events can end up handled out of sequence. Where you need ordered delivery at high throughput, lean on [Channels](https://v2.tauri.app/develop/calling-frontend/#channels) rather than the event system.

#### Framework-specific Cleanup Examples

With any frontend framework, tearing down event listeners as a component unmounts is important so you don't leak memory or end up with duplicate handlers.

- React
- Vue

- [React event listener cleanup](react-event-listener-cleanup.md): Open when registering and cleaning up a listener in React.
- [Vue event listener cleanup](vue-event-listener-cleanup.md): Open when registering and cleaning up a listener in Vue.
- [Svelte event listener cleanup](svelte-event-listener-cleanup.md): Open when registering and cleaning up a listener in Svelte.

Tauri also ships a helper that subscribes to an event just a single time:

```js
import { once } from '@tauri-apps/api/event';
import { getCurrentWebviewWindow } from '@tauri-apps/api/webviewWindow';

once('ready', (event) => {});

const appWebview = getCurrentWebviewWindow();
appWebview.once('ready', () => {});
```

