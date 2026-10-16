# Android command argument models

Open when defining annotated Kotlin classes for mobile command arguments.

[Topic context](mobile-command-argument-models.md)

### Command Arguments

Command arguments arrive serialized, and the handset plugin decodes them through the `Invoke::parseArgs` function, which accepts a class that models the argument object.

### Android

Under Android, you express the arguments as a class carrying the `@app.tauri.annotation.InvokeArg` annotation, and any nested objects need that annotation too:

```kotlin
import app.tauri.plugin.Plugin
import app.tauri.plugin.Invoke
import app.tauri.plugin.JSObject

import android.app.Activity
import android.webkit.WebView
import app.tauri.annotation.Command
import app.tauri.annotation.InvokeArg
import app.tauri.annotation.TauriPlugin

@InvokeArg
internal class OpenAppArgs {
  lateinit var name: String
  var timeout: Int? = null
}

@InvokeArg
internal class OpenArgs {
  lateinit var requiredArg: String
  var allowEdit: Boolean = false
  var quality: Int = 100
  var app: OpenAppArgs? = null
}

@TauriPlugin
class ExamplePlugin(private val activity: Activity): Plugin(activity) {
  @Command
  fun openCamera(invoke: Invoke) {
    val args = invoke.parseArgs(OpenArgs::class.java)
    val result = JSObject()
    result.put("path", "/path/to/photo.jpg")
    invoke.resolve(result)
  }
}
```
