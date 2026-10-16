# Request mobile permissions

Open when checking or requesting mobile plugin permissions from TypeScript or Rust.

[Topic context](mobile-plugin-permissions-events.md)

On its own, Tauri wires up a pair of commands for the plugin: `checkPermissions` along with `requestPermissions`. Either one is callable straight from JavaScript or Rust:

- JavaScript
- Rust

```ts
import { invoke, PermissionState } from '@tauri-apps/api/core'

interface Permissions {
  postNotification: PermissionState
}

// check permission state
const permission = await invoke<Permissions>('plugin:<plugin-name>|checkPermissions')

if (permission.postNotification === 'prompt-with-rationale') {
  // show information to the user about why permission is needed
}

// request permission
if (permission.postNotification.startsWith('prompt')) {
  const state = await invoke<Permissions>('plugin:<plugin-name>|requestPermissions', { permissions: ['postNotification'] })
}
```

```rust
use serde::{Serialize, Deserialize};
use tauri::{plugin::PermissionState, Runtime};

#[derive(Deserialize)]
#[serde(rename_all = "camelCase")]
struct PermissionResponse {
  pub post_notification: PermissionState,
}

#[derive(Serialize)]
#[serde(rename_all = "camelCase")]
struct RequestPermission {
  permissions: Vec<&'static str>,
}

impl<R: Runtime> Notification<R> {
  pub fn request_post_notification_permission(&self) -> crate::Result<PermissionState> {
    self.0
      .run_mobile_plugin::<PermissionResponse>("requestPermissions", RequestPermission { permissions: vec!["postNotification"] })
      .map(|r| r.post_notification)
      .map_err(Into::into)
  }

  pub fn check_permissions(&self) -> crate::Result<PermissionResponse> {
    self.0
      .run_mobile_plugin::<PermissionResponse>("checkPermissions", ())
      .map_err(Into::into)
  }
}
```

The Rust methods extend the plugin template wrapper around PluginHandle. Use the [native permission implementations](mobile-permission-implementations.md) and replace the plugin name before invoking them.
