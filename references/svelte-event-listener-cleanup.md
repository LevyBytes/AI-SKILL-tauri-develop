# Svelte event listener cleanup

Open when releasing a Tauri event listener during Svelte component cleanup.

[Topic context](frontend-to-rust-listener-lifecycle.md)

```svelte
<script lang="ts">
import { listen } from '@tauri-apps/api/event';

let progress = $state(0);

$effect(() => {
  const unlistenPromise = listen<number>(
    'download-progress',
    (event) => {
      progress = event.payload;
    }
  );

  return () => {
    unlistenPromise.then((unlisten) => unlisten());
  };
});
</script>

<div>Download progress: {progress}%</div>
```

Also applies to [Rust-to-frontend events](rust-to-frontend-listener-lifecycle.md).
