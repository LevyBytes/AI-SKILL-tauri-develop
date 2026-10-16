# Mock frontend windows

Open when testing current and multiple webview window identities.

[Topic context](frontend-ipc-event-mocks.md)

### Windows

Use mockWindows to supply the current window identity. In @tauri-apps/api 2.12.2 the additional label arguments are ignored; getAllWebviewWindows requires a separate IPC response for plugin:window|get_all_windows.

```javascript
import { afterEach, expect, test } from 'vitest';
import { mockIPC, mockWindows, clearMocks } from '@tauri-apps/api/mocks';

afterEach(() => clearMocks());

test('window identities', async () => {
  mockWindows('main');
  mockIPC((cmd) => {
    if (cmd === 'plugin:window|get_all_windows') {
      return ['main', 'second', 'third'];
    }
    throw new Error(`Unexpected command: ${cmd}`);
  });
  const { getCurrentWebviewWindow, getAllWebviewWindows } =
    await import('@tauri-apps/api/webviewWindow');
  expect(getCurrentWebviewWindow()).toHaveProperty('label', 'main');
  const windows = await getAllWebviewWindows();
  expect(windows.map((w) => w.label)).toEqual(['main', 'second', 'third']);
});
```

---
