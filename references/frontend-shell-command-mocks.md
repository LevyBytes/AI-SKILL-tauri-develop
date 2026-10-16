# Mock shell commands

Open when testing shell or sidecar command results without launching a process.

[Topic context](frontend-ipc-event-mocks.md)

The v2 shell plugin returns a completed result from execute(), while spawn() receives an onEvent Channel. Mock these separately. This example intercepts IPC; it starts no native process.

```javascript
import { mockIPC, clearMocks } from '@tauri-apps/api/mocks';
import { Command } from '@tauri-apps/plugin-shell';
import { afterEach, expect, test, vi } from 'vitest';

afterEach(() => clearMocks());

test('shell results and streamed output', async () => {
  let processEvents;
  mockIPC((cmd, args) => {
    if (cmd === 'plugin:shell|execute') {
      return { code: 0, signal: null, stdout: 'complete', stderr: '' };
    }
    if (cmd === 'plugin:shell|spawn') {
      processEvents = args.onEvent;
      return 123;
    }
    throw new Error(`Unexpected command: ${cmd}`);
  });
  const output = await Command.sidecar('my-sidecar').execute();
  expect(output.stdout).toBe('complete');
  const command = Command.sidecar('my-sidecar');
  const stdout = vi.fn();
  const close = vi.fn();
  command.stdout.on('data', stdout);
  command.on('close', close);
  const child = await command.spawn();
  expect(child.pid).toBe(123);
  processEvents.onmessage({ event: 'Stdout', payload: 'streamed' });
  processEvents.onmessage({ event: 'Terminated', payload: { code: 0, signal: null } });
  expect(stdout).toHaveBeenCalledWith('streamed');
  expect(close).toHaveBeenCalledWith({ code: 0, signal: null });
});
```
