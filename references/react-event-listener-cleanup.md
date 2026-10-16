# React event listener cleanup

Open when releasing a Tauri event listener during React component cleanup.

[Topic context](frontend-to-rust-listener-lifecycle.md)

```tsx
import { useEffect, useState } from 'react';
import { listen } from '@tauri-apps/api/event';

function DownloadTracker() {
  const [progress, setProgress] = useState(0);

  useEffect(() => {
    const unlisten = listen<number>('download-progress', (event) => {
      setProgress(event.payload);
    });

    return () => {
      unlisten.then((fn) => fn());
    };
  }, []);

  return <div>Download progress: {progress}%</div>;
}
```

Also applies to [Rust-to-frontend events](rust-to-frontend-listener-lifecycle.md).
