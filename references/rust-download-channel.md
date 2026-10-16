# Rust download channel

Open when sending ordered download events from Rust through a typed channel.

[Topic context](frontend-channels-javascript-evaluation.md)

### Channels

As a basic, app-wide two-way channel of communication, the event system is built for simplicity. Because it works by running JavaScript directly behind the scenes, it can fall short when you need to ship a lot of data.

Channels, by contrast, are built for speed and guarantee ordered delivery. Internally they back streaming work like download progress, child-process output, and WebSocket traffic.

Let's recast the download command from earlier so it relies on channels in place of the event system:

src-tauri/src/lib.rs

```rust
use tauri::{AppHandle, ipc::Channel};
use serde::Serialize;

#[derive(Clone, Serialize)]
#[serde(rename_all = "camelCase", rename_all_fields = "camelCase", tag = "event", content = "data")]
enum DownloadEvent<'a> {
  Started {
    url: &'a str,
    download_id: usize,
    content_length: usize,
  },
  Progress {
    download_id: usize,
    chunk_length: usize,
  },
  Finished {
    download_id: usize,
  },
}

#[tauri::command]
fn download(app: AppHandle, url: String, on_event: Channel<DownloadEvent>) {
  let content_length = 1000;
  let download_id = 1;

  on_event.send(DownloadEvent::Started {
    url: &url,
    download_id,
    content_length,
  }).unwrap();

  for chunk_length in [15, 150, 35, 500, 300] {
    on_event.send(DownloadEvent::Progress {
      download_id,
      chunk_length,
    }).unwrap();
  }

  on_event.send(DownloadEvent::Finished { download_id }).unwrap();
}
```

Receive these events with the [matching frontend channel](frontend-download-channel.md).
