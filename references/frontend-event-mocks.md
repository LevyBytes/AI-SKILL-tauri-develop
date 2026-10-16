# Mock frontend events

Open when testing event handlers with mockIPC and cleaning up mocked events.

[Topic context](frontend-ipc-event-mocks.md)

### Mocking Events

[Since `2.7.0`](https://v2.tauri.app/release/tauri/v2.7.0)

The [Event System](https://v2.tauri.app/develop/calling-frontend/#event-system) is partly covered too, letting you mimic events your Rust code would emit by setting the `shouldMockEvents` option:

```javascript
import { mockIPC, clearMocks } from '@tauri-apps/api/mocks';
import { emit, listen } from '@tauri-apps/api/event';
import { afterEach, expect, test, vi } from 'vitest';

afterEach(() => clearMocks());

test('mocked event', async () => {
  mockIPC(() => {}, { shouldMockEvents: true });
  const handler = vi.fn();
  const unlisten = await listen('test-event', handler);
  await emit('test-event', { foo: 'bar' });
  expect(handler).toHaveBeenCalledWith({
    event: 'test-event', payload: { foo: 'bar' },
  });
  unlisten();
});
```

`emitTo` and `emit_filter` are **not** supported yet.
