# Deliver mobile plugin events

Open when emitting mobile plugin events and subscribing from the frontend.

[Topic context](mobile-plugin-permissions-events.md)

### Plugin Events

At any moment a plugin may dispatch events by calling the `trigger` function:

- Android
- iOS

```kotlin
import android.app.Activity
import android.webkit.WebView
import android.content.Intent
import app.tauri.plugin.Plugin
import app.tauri.plugin.Invoke
import app.tauri.plugin.JSObject
import app.tauri.annotation.TauriPlugin
import app.tauri.annotation.Command

@TauriPlugin
class ExamplePlugin(private val activity: Activity): Plugin(activity) {
    override fun load(webView: WebView) {
      trigger("load", JSObject())
    }

    override fun onNewIntent(intent: Intent) {
      // handle new intent event
      if (intent.action == Intent.ACTION_VIEW) {
        val data = intent.data.toString()
        val event = JSObject()
        event.put("data", data)
        trigger("newIntent", event)
      }
    }

    @Command
    fun openCamera(invoke: Invoke) {
      val payload = JSObject()
      payload.put("open", true)
      trigger("camera", payload)
      invoke.resolve()
    }
}
```

```swift
import Tauri
import WebKit

class ExamplePlugin: Plugin {
  @objc public override func load(webview: WKWebView) {
    trigger("load", data: [:])
  }

  @objc public func openCamera(_ invoke: Invoke) {
    trigger("camera", data: ["open": true])
    invoke.resolve()
  }
}
```

From the NPM package, you reach those helper functions through the [`addPluginListener`](https://v2.tauri.app/reference/javascript/api/namespacecore/#addpluginlistener) helper:

```ts
import { addPluginListener, PluginListener } from '@tauri-apps/api/core';

export async function onRequest(
  handler: (url: string) => void
): Promise<PluginListener> {
  return await addPluginListener(
    '<plugin-name>',
    'event-name',
    handler
  );
}
```

---
