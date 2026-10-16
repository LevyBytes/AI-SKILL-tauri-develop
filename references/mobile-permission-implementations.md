# Implement mobile permissions

Open when declaring Android permission aliases or implementing iOS permission requests.

[Topic context](mobile-plugin-permissions-events.md)

### Permissions

When a plugin needs permissions granted by the end user, Tauri streamlines both checking and asking for them.

- Android
- iOS

Begin by spelling out which permissions you require plus an alias that names each group in code; all of this lives inside the `TauriPlugin` annotation:

```kotlin
import android.app.Activity
import android.Manifest
import app.tauri.plugin.Plugin
import app.tauri.annotation.TauriPlugin
import app.tauri.annotation.Permission

@TauriPlugin(
  permissions = [
    Permission(strings = [Manifest.permission.POST_NOTIFICATIONS], alias = "postNotification")
  ]
)
class ExamplePlugin(private val activity: Activity): Plugin(activity) { }
```

Start by overriding the two functions `checkPermissions` and `requestPermissions`:

```swift
import Tauri

class ExamplePlugin: Plugin {
  @objc public override func checkPermissions(_ invoke: Invoke) {
    invoke.resolve(["postNotification": "prompt"])
  }

  @objc public override func requestPermissions(_ invoke: Invoke) {
    // request permissions here
    // then resolve the request
    invoke.resolve(["postNotification": "granted"])
  }
}
```

The iOS responses illustrate the command contract. Replace the permission-request placeholder with the platform permission API and report its actual result.
