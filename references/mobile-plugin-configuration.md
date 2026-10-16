# Configure mobile plugins

Open when reading plugin configuration in Kotlin or Swift.

[Topic context](mobile-plugin-initialization-configuration.md)

### Plugin Configuration

See the [Plugin Configuration section](https://v2.tauri.app/develop/plugins/#plugin-configuration) within the Plugin Development guide if you want a fuller account of building plugin configurations.

On the handset side, the plugin instance exposes a getter that returns its configuration:

- Android
- iOS

```kotlin
import app.tauri.plugin.Plugin

import android.app.Activity
import android.webkit.WebView
import app.tauri.annotation.TauriPlugin
import app.tauri.annotation.InvokeArg

@InvokeArg
class Config {
    var timeout: Int? = 3000
}

@TauriPlugin
class ExamplePlugin(private val activity: Activity): Plugin(activity) {
  private var timeout: Int? = 3000

  override fun load(webView: WebView) {
    getConfig(Config::class.java).let {
       this.timeout = it.timeout
    }
  }
}
```

```swift
import Tauri
import WebKit

struct Config: Decodable {
  let timeout: Int?
}

class ExamplePlugin: Plugin {
  var timeout: Int? = 3000

  @objc public override func load(webview: WKWebView) {
    do {
      let config = try parseConfig(Config.self)
      self.timeout = config.timeout
    } catch {}
  }
}
```
