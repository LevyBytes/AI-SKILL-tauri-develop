# Vue event listener cleanup

Open when releasing a Tauri event listener during Vue component cleanup.

[Topic context](frontend-to-rust-listener-lifecycle.md)

```vue
<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import { listen } from '@tauri-apps/api/event';

const progress = ref(0);
let unlistenPromise: Promise<() => void> | undefined;

onMounted(() => {
  unlistenPromise = listen<number>('download-progress', (event) => {
    progress.value = event.payload;
  });
});

onUnmounted(() => {
  unlistenPromise?.then((fn) => fn());
});
</script>

<template>
  <div>Download progress: {{ progress }}%</div>
</template>
```

Also applies to [Rust-to-frontend events](rust-to-frontend-listener-lifecycle.md).
