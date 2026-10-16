# Mobile load and intent lifecycle

[Topic context](mobile-plugin-development-index.md) / [Previous source section](mobile-plugin-initialization-configuration.md) / [Next source section](mobile-command-argument-models.md)

Context: Tauri v2: Mobile load and intent lifecycle. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-lifecycle-events"></a> <a id="source-load"></a> <a id="source-onnewintent"></a>

### Lifecycle Events

A plugin is able to tie into a handful of lifecycle events:

- load: fires as the plugin is brought into the web view
- onNewIntent: Android exclusively, triggered when the activity gets re-launched

Further [lifecycle events for plugins](https://v2.tauri.app/develop/plugins/#lifecycle-events) are documented in the Plugin Development guide as well.

### load

- **When**: at the moment the plugin gets brought into the web view
- **Why**: to run the plugin's startup code

- Android
- iOS

```kotlin
import app.tauri.plugin.Plugin

import android.app.Activity
import android.webkit.WebView
import app.tauri.annotation.TauriPlugin

@TauriPlugin
class ExamplePlugin(private val activity: Activity): Plugin(activity) {
  override fun load(webView: WebView) {
    // perform plugin setup here
  }
}
```

```swift
import Tauri
import WebKit

class ExamplePlugin: Plugin {
  @objc public override func load(webview: WKWebView) {
    let timeout = self.config["timeout"] as? Int ?? 30
  }
}
```

### onNewIntent

**Note**: Android is the only platform where this exists.

- **When**: upon the activity being re-launched. Consult [Activity#onNewIntent](<https://developer.android.com/reference/android/app/Activity#onNewIntent(android.content.Intent)>) for further detail.
- **Why**: to cope with the app relaunching, for instance after a tapped notification or a followed deep link.

```kotlin
import app.tauri.plugin.Plugin

import android.app.Activity
import android.content.Intent
import app.tauri.annotation.TauriPlugin

@TauriPlugin
class ExamplePlugin(private val activity: Activity): Plugin(activity) {
  override fun onNewIntent(intent: Intent) {
    // handle new intent event
  }
}
```

