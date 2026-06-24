# Mocking

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Tests; Mock Tauri APIs.

## Tests

### Tests

Both unit and integration tests are possible in Tauri through a mock runtime. While that mock runtime is active, the platform's native webview libraries stay unused. [See more about the mock runtime here](https://v2.tauri.app/develop/tests/mocking/).

For end-to-end coverage, Tauri leans on the WebDriver protocol. This applies to desktop as well as mobile targets, with the single exception of macOS, where no desktop WebDriver client exists. [See more about WebDriver support here](https://v2.tauri.app/develop/tests/webdriver/).

For convenience there is [tauri-action](https://github.com/tauri-apps/tauri-action), which drives GitHub actions, though Tauri pairs fine with whatever CI/CD runner you prefer—provided the host carries every library each platform needs in order to compile.

---

## Mock Tauri APIs

### Mock Tauri APIs

As you build out your frontend tests, it's routine to stand up a “fake” Tauri environment that fakes windows or catches IPC traffic—what people call _mocking_. To smooth that process, the [`@tauri-apps/api/mocks`](https://v2.tauri.app/reference/javascript/api/namespacemocks/) module ships a few convenient utilities:

### IPC Requests

The most frequent need is catching IPC requests, which proves useful across several scenarios:

- Confirm that the right backend calls actually fire
- Fake varied return values from backend functions

For grabbing IPC requests, Tauri exposes the mockIPC function. Full details on that particular API live [here](https://v2.tauri.app/reference/javascript/api/namespacemocks/#mockipc).

### Mocking Commands for `invoke`

```
import { beforeAll, expect, test } from "vitest";import { randomFillSync } from "crypto";
import { mockIPC } from "@tauri-apps/api/mocks";import { invoke } from "@tauri-apps/api/core";
// jsdom doesn't come with a WebCrypto implementationbeforeAll(() => {  Object.defineProperty(window, 'crypto', {    value: {      // @ts-ignore      getRandomValues: (buffer) => {        return randomFillSync(buffer);      },    },  });});

test("invoke simple", async () => {  mockIPC((cmd, args) => {    // simulated rust command called "add" that just adds two numbers    if(cmd === "add") {      return (args.a as number) + (args.b as number);    }  });});
```

Occasionally you need richer detail about an IPC call—was the command run once, several times, or never at all? Pairing [`mockIPC()`](https://v2.tauri.app/reference/javascript/api/namespacemocks/#mockipc) with additional spying and mocking utilities lets you check exactly that:

```
import { beforeAll, expect, test, vi } from "vitest";import { randomFillSync } from "crypto";
import { mockIPC } from "@tauri-apps/api/mocks";import { invoke } from "@tauri-apps/api/core";
// jsdom doesn't come with a WebCrypto implementationbeforeAll(() => {  Object.defineProperty(window, 'crypto', {    value: {      // @ts-ignore      getRandomValues: (buffer) => {        return randomFillSync(buffer);      },    },  });});

test("invoke", async () => {  mockIPC((cmd, args) => {    // simulated rust command called "add" that just adds two numbers    if(cmd === "add") {      return (args.a as number) + (args.b as number);    }  });
  // we can use the spying tools provided by vitest to track the mocked function  const spy = vi.spyOn(window.__TAURI_INTERNALS__, "invoke");
  expect(invoke("add", { a: 12, b: 15 })).resolves.toBe(27);  expect(spy).toHaveBeenCalled();});
```

When the target of your IPC mock is a sidecar or shell command, capture the event handler's ID at the moment `spawn()` or `execute()` runs, then reuse that ID to push out the events the backend would normally return:

```
mockIPC(async (cmd, args) => {  if (args.message.cmd === 'execute') {    const eventCallbackId = `_${args.message.onEventFn}`;    const eventEmitter = window[eventCallbackId];
    // 'Stdout' event can be called multiple times    eventEmitter({      event: 'Stdout',      payload: 'some data sent from the process',    });
    // 'Terminated' event must be called at the end to resolve the promise    eventEmitter({      event: 'Terminated',      payload: {        code: 0,        signal: 'kill',      },    });  }});
```

### Mocking Events

[Since `2.7.0`](https://v2.tauri.app/release/tauri/v2.7.0)

The [Event System](https://v2.tauri.app/develop/calling-frontend/#event-system) is partly covered too, letting you mimic events your Rust code would emit by setting the `shouldMockEvents` option:

```
import { mockIPC, clearMocks } from '@tauri-apps/api/mocks';import { emit, listen } from '@tauri-apps/api/event';import { afterEach, expect, test, vi } from 'vitest';
test('mocked event', () => {  mockIPC(() => {}, { shouldMockEvents: true }); // enable event mocking
  const eventHandler = vi.fn();  listen('test-event', eventHandler);
  emit('test-event', { foo: 'bar' });  expect(eventHandler).toHaveBeenCalledWith({    event: 'test-event',    payload: { foo: 'bar' },  });});
```

`emitTo` and `emit_filter` are **not** supported yet.

### Windows

Certain logic is tied to a particular window (think of a splash screen window), which means your tests have to stand in for several windows at once. The [`mockWindows()`](https://v2.tauri.app/reference/javascript/api/namespacemocks/#mockwindows) method exists to fabricate window labels for that purpose. Its first string names the “current” window—the one your JavaScript assumes it lives in—while every remaining string registers as an extra window.

```
import { beforeAll, expect, test } from 'vitest';import { randomFillSync } from 'crypto';
import { mockWindows } from '@tauri-apps/api/mocks';
// jsdom doesn't come with a WebCrypto implementationbeforeAll(() => {  Object.defineProperty(window, 'crypto', {    value: {      // @ts-ignore      getRandomValues: (buffer) => {        return randomFillSync(buffer);      },    },  });});
test('invoke', async () => {  mockWindows('main', 'second', 'third');
  const { getCurrent, getAll } = await import('@tauri-apps/api/webviewWindow');
  expect(getCurrent()).toHaveProperty('label', 'main');  expect(getAll().map((w) => w.label)).toEqual(['main', 'second', 'third']);});
```

---
