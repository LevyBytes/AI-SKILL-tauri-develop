# Mock frontend commands

Open when setting up frontend tests and intercepting command invocations.

[Topic context](frontend-ipc-event-mocks.md)

# Mocking

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Tests; Mock Tauri APIs.

## Tests

### Tests

Both unit and integration tests are possible in Tauri through a mock runtime. While that mock runtime is active, the platform's native webview libraries stay unused. [See more about the mock runtime here](https://v2.tauri.app/develop/tests/mocking/).

End-to-end tests can drive the application through WebDriver. Direct tauri-driver operation supports Windows and Linux; the WebdriverIO Tauri service also provides an embedded driver for macOS. See the [WebDriver overview](https://v2.tauri.app/develop/tests/webdriver/) for the supported setup.

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

```javascript
import { afterEach, beforeAll, expect, test } from "vitest";
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


test("invoke simple", async () => {
  mockIPC((cmd, args) => {
    // simulated rust command called "add" that just adds two numbers
    if(cmd === "add") {
      return Number(args?.a) + Number(args?.b);
    }
  });
  await expect(invoke("add", { a: 12, b: 15 })).resolves.toBe(27);
});
```
