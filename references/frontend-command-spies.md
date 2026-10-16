# Spy on command invocations

Open when asserting which IPC commands a frontend test invokes.

[Topic context](frontend-ipc-event-mocks.md)

Occasionally you need richer detail about an IPC call—was the command run once, several times, or never at all? Pairing [`mockIPC()`](https://v2.tauri.app/reference/javascript/api/namespacemocks/#mockipc) with additional spying and mocking utilities lets you check exactly that:

```javascript
import { afterEach, beforeAll, expect, test, vi } from "vitest";
import { randomFillSync } from "crypto";

import { mockIPC, clearMocks } from "@tauri-apps/api/mocks";
import { invoke } from "@tauri-apps/api/core";

afterEach(() => clearMocks());

// jsdom doesn't come with a WebCrypto implementation
beforeAll(() => {
  Object.defineProperty(window, 'crypto', {
    value: {
      // @ts-ignore
      getRandomValues: (buffer) => {
        return randomFillSync(buffer);
      },
    },
  });
});


test("invoke", async () => {
  mockIPC((cmd, args) => {
    // simulated rust command called "add" that just adds two numbers
    if(cmd === "add") {
      return Number(args?.a) + Number(args?.b);
    }
  });

  // we can use the spying tools provided by vitest to track the mocked function
  const spy = vi.spyOn(window.__TAURI_INTERNALS__, "invoke");

  await expect(invoke("add", { a: 12, b: 15 })).resolves.toBe(27);
  expect(spy).toHaveBeenCalled();
});
```
