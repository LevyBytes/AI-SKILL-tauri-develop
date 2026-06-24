# Develop Mobile

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

### Mobile Plugin Development

A plugin may host native handset code authored in Kotlin (or Java) alongside Swift. The starter template ships an Android library module built on Kotlin plus a Swift package, and the latter carries a sample mobile command that demonstrates firing it off from Rust.

### Initialize Plugin Project

Walk through the procedure laid out in the [Plugin Development guide](https://v2.tauri.app/develop/plugins/#initialize-plugin-project) to spin up a fresh plugin project.

Already have a plugin and want to extend it with Android or iOS support? Reach for `plugin android init` and `plugin ios init`, which scaffold the handset library projects and walk you through whatever edits are required.

Out of the box, the template divides a plugin's logic across a pair of distinct modules: `desktop.rs` and `mobile.rs`.

On desktop the feature is built straight in Rust, whereas the handset path dispatches a message into the native mobile layer to run a routine and return its result. Any logic the two sides have in common belongs in `lib.rs`:

src/lib.rs

```
use tauri::Runtime;
impl<R: Runtime> <plugin-name><R> {  pub fn do_something(&self) {    // do something that is a shared implementation between desktop and mobile  }}
```

Doing it this way makes it easy to expose one API that both commands and plain Rust code can reach for.

### Develop an Android Plugin

On Android a Tauri plugin takes the form of a Kotlin class subclassing `app.tauri.plugin.Plugin` and carrying the `app.tauri.annotation.TauriPlugin` annotation. Any method tagged with `app.tauri.annotation.Command` becomes reachable from Rust or JavaScript.

Although Tauri defaults to Kotlin for the Android side, Java remains an option when that suits you better. Once the plugin exists, right-click its Kotlin plugin class within Android Studio and pick “Convert Kotlin file to Java file” from the menu; Android Studio then steps you through converting the project to Java.

### Develop an iOS Plugin

For iOS, a Tauri plugin is written as a Swift class that derives from the `Plugin` class supplied by the `Tauri` package. Whenever a function bears the `@objc` attribute together with the `(_ invoke: Invoke)` parameter (say `@objc private func download(_ invoke: Invoke) { }`), Rust or JavaScript can invoke it.

Because the plugin is structured as a [Swift package](https://www.swift.org/package-manager/), its package manager handles dependency management for you.

### Plugin Configuration

See the [Plugin Configuration section](https://v2.tauri.app/develop/plugins/#plugin-configuration) within the Plugin Development guide if you want a fuller account of building plugin configurations.

On the handset side, the plugin instance exposes a getter that returns its configuration:

- Android
- iOS

```
import android.app.Activityimport android.webkit.WebViewimport app.tauri.annotation.TauriPluginimport app.tauri.annotation.InvokeArg
@InvokeArgclass Config {    var timeout: Int? = 3000}
@TauriPluginclass ExamplePlugin(private val activity: Activity): Plugin(activity) {  private var timeout: Int? = 3000
  override fun load(webView: WebView) {    getConfig(Config::class.java).let {       this.timeout = it.timeout    }  }}
```

```
struct Config: Decodable {  let timeout: Int?}
class ExamplePlugin: Plugin {  var timeout: Int? = 3000
  @objc public override func load(webview: WKWebView) {    do {      let config = try parseConfig(Config.self)      self.timeout = config.timeout    } catch {}  }}
```

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

```
import android.app.Activityimport android.webkit.WebViewimport app.tauri.annotation.TauriPlugin
@TauriPluginclass ExamplePlugin(private val activity: Activity): Plugin(activity) {  override fun load(webView: WebView) {    // perform plugin setup here  }}
```

```
class ExamplePlugin: Plugin {  @objc public override func load(webview: WKWebView) {    let timeout = self.config["timeout"] as? Int ?? 30  }}
```

### onNewIntent

**Note**: Android is the only platform where this exists.

- **When**: upon the activity being re-launched. Consult [Activity#onNewIntent](<https://developer.android.com/reference/android/app/Activity#onNewIntent(android.content.Intent)>) for further detail.
- **Why**: to cope with the app relaunching, for instance after a tapped notification or a followed deep link.

```
import android.app.Activityimport android.content.Intentimport app.tauri.annotation.TauriPlugin
@TauriPluginclass ExamplePlugin(private val activity: Activity): Plugin(activity) {  override fun onNewIntent(intent: Intent) {    // handle new intent event  }}
```

### Adding Mobile Commands

Within each handset project sits a plugin class, and that is where you declare commands the Rust code is able to call:

- Android
- iOS

```
import android.app.Activityimport app.tauri.annotation.Commandimport app.tauri.annotation.TauriPlugin
@TauriPluginclass ExamplePlugin(private val activity: Activity): Plugin(activity) {  @Command  fun openCamera(invoke: Invoke) {    val ret = JSObject()    ret.put("path", "/path/to/photo.jpg")    invoke.resolve(ret)  }}
```

Should you reach for a Kotlin `suspend` function, a custom coroutine scope becomes necessary

```
import android.app.Activityimport app.tauri.annotation.Commandimport app.tauri.annotation.TauriPlugin
// Change to Dispatchers.IO if it is intended for fetching dataval scope = CoroutineScope(Dispatchers.Default + SupervisorJob())
@TauriPluginclass ExamplePlugin(private val activity: Activity): Plugin(activity) {  @Command  fun openCamera(invoke: Invoke) {    scope.launch {      openCameraInner(invoke)    }  }
  private suspend fun openCameraInner(invoke: Invoke) {    val ret = JSObject()    ret.put("path", "/path/to/photo.jpg")    invoke.resolve(ret)  }}
```

```
class ExamplePlugin: Plugin {  @objc public func openCamera(_ invoke: Invoke) throws {    invoke.resolve(["path": "/path/to/photo.jpg"])  }}
```

Lean on the [`tauri::plugin::PluginHandle`](https://docs.rs/tauri/2.0.0/tauri/plugin/struct.PluginHandle.html) when you need to fire a mobile command from the Rust side:

```
use std::path::PathBuf;use serde::{Deserialize, Serialize};use tauri::Runtime;
#[derive(Serialize)]#[serde(rename_all = "camelCase")]pub struct CameraRequest {  quality: usize,  allow_edit: bool,}
#[derive(Deserialize)]pub struct Photo {  path: PathBuf,}

impl<R: Runtime> <plugin-name;pascal-case><R> {  pub fn open_camera(&self, payload: CameraRequest) -> crate::Result<Photo> {    self      .0      .run_mobile_plugin("openCamera", payload)      .map_err(Into::into)  }}
```

### Command Arguments

Command arguments arrive serialized, and the handset plugin decodes them through the `Invoke::parseArgs` function, which accepts a class that models the argument object.

### Android

Under Android, you express the arguments as a class carrying the `@app.tauri.annotation.InvokeArg` annotation, and any nested objects need that annotation too:

```
import android.app.Activityimport android.webkit.WebViewimport app.tauri.annotation.Commandimport app.tauri.annotation.InvokeArgimport app.tauri.annotation.TauriPlugin
@InvokeArginternal class OpenAppArgs {  lateinit var name: String  var timeout: Int? = null}
@InvokeArginternal class OpenArgs {  lateinit var requiredArg: String  var allowEdit: Boolean = false  var quality: Int = 100  var app: OpenAppArgs? = null}
@TauriPluginclass ExamplePlugin(private val activity: Activity): Plugin(activity) {  @Command  fun openCamera(invoke: Invoke) {    val args = invoke.parseArgs(OpenArgs::class.java)  }}
```

### iOS

For iOS, you model the arguments with a class conforming to `Decodable`, and nested objects likewise have to adopt the Decodable protocol:

```
class OpenAppArgs: Decodable {  let name: String  var timeout: Int?}
class OpenArgs: Decodable {  let requiredArg: String  var allowEdit: Bool?  var quality: UInt8?  var app: OpenAppArgs?}
class ExamplePlugin: Plugin {  @objc public func openCamera(_ invoke: Invoke) throws {    let args = try invoke.parseArgs(OpenArgs.self)
    invoke.resolve(["path": "/path/to/photo.jpg"])  }}
```

### Calling Rust From Mobile Plugins

Writing plugin logic in Rust is frequently the better choice, both for speed and for reuse. Tauri offers no built-in path to reach Rust from inside your plugin code, yet JNI on Android and FFI on iOS let plugins invoke shared code even while the app's WebView sits suspended.

### Android

Inside the `Cargo.toml` belonging to your plugin, list the jni crate among the dependencies:

```
[target.'cfg(target_os = "android")'.dependencies]jni = "0.21"
```

Pull in the application library statically and declare the native functions within your Kotlin code. Here the Kotlin class happens to be `com.example.HelloWorld`, and the Rust side has to cite that complete package name.

```
private const val TAG = "MyPlugin"
init {  try {    // Load the native library (libapp_lib.so)    // This is the shared library built by Cargo with crate-type = ["cdylib"]    System.loadLibrary("app_lib")    Log.d(TAG, "Successfully loaded libapp_lib.so")  } catch (e: UnsatisfiedLinkError) {    Log.e(TAG, "Failed to load libapp_lib.so", e)    throw e  }}
external fun helloWorld(name: String): String?
```

Next, over in your plugin's Rust code, declare the function that JNI expects to find. Names follow the `Java_package_class_method` shape, so for the class shown earlier it works out to `Java_com_example_HelloWorld_helloWorld`, which is what our `helloWorld` method ends up calling:

```
#[cfg(target_os = "android")]#[no_mangle]pub extern "system" fn Java_com_example_HelloWorld_helloWorld(    mut env: JNIEnv,    _class: JClass,    name: JString,) -> jstring {    log::debug!("Calling JNI Hello World!");    let result = format!("Hello, {}!", name);
    match env.new_string(result) {        Ok(jstr) => jstr.into_raw(),        Err(e) => {            log::error!("Failed to create JString: {}", e);            std::ptr::null_mut()        }    }}
```

### iOS

Since iOS relies purely on plain C FFI, no extra dependencies come into play. Place the hook inside your Swift code together with whatever teardown is needed. You may give these functions any valid name, provided each one carries the `@_silgen_name(FFI_FUNC)` annotation, in which FFI_FUNC stands for the function name that Rust will call:

```
@_silgen_name("hello_world_ffi")private static func helloWorldFFI(_ name: UnsafePointer<CChar>) -> UnsafeMutablePointer<CChar>?
@_silgen_name("free_hello_result_ffi")private static func freeHelloResult(_ result: UnsafeMutablePointer<CChar>)
static func helloWorld(name: String) -> String? {  // Call Rust FFI  let resultPtr = name.withCString({ helloWorldFFI($0) })
  // Convert C string to Swift String  let result = String(cString: resultPtr)
  // Free the C string  freeHelloResult(resultPtr)
  return result}
```

Now build out the Rust half. Be sure the `extern` functions here line up with the `@_silgen_name` annotations declared over on the Swift side:

```
#[no_mangle]pub unsafe extern "C" fn hello_world_ffi(c_name: *const c_char) -> *mut c_char {    let name = match CStr::from_ptr(c_name).to_str() {        Ok(s) => s,        Err(e) => {            log::error!("[iOS FFI] Failed to convert C string: {}", e);            return std::ptr::null_mut();        }    };
    let result = format!("Hello, {}!", name);
    match CString::new(result) {        Ok(c_str) => c_str.into_raw(),        Err(e) => {            log::error!("[iOS FFI] Failed to create C string: {}", e);            std::ptr::null_mut()        }    }}
#[no_mangle]pub unsafe extern "C" fn free_hello_result_ffi(result: *mut c_char) {    if !result.is_null() {        drop(CString::from_raw(result));    }}
```

### Android 16KB Memory Pages

Google is in the process of mandating 16KB memory pages for every new Android app submission. Compiling against NDK version 28 or later ought to emit bundles that already satisfy the rule, yet if you are stuck on an older NDK or the produced files end up not 16KB aligned, dropping the following into `.cargo/config.toml` signals it to `rustc`:

```
[target.aarch64-linux-android]rustflags = ["-C", "link-arg=-Wl,-z,max-page-size=16384"]
```

### Permissions

When a plugin needs permissions granted by the end user, Tauri streamlines both checking and asking for them.

- Android
- iOS

Begin by spelling out which permissions you require plus an alias that names each group in code; all of this lives inside the `TauriPlugin` annotation:

```
@TauriPlugin(  permissions = [    Permission(strings = [Manifest.permission.POST_NOTIFICATIONS], alias = "postNotification")  ])class ExamplePlugin(private val activity: Activity): Plugin(activity) { }
```

Start by overriding the two functions `checkPermissions` and `requestPermissions`:

```
class ExamplePlugin: Plugin {  @objc open func checkPermissions(_ invoke: Invoke) {    invoke.resolve(["postNotification": "prompt"])  }
  @objc public override func requestPermissions(_ invoke: Invoke) {    // request permissions here    // then resolve the request    invoke.resolve(["postNotification": "granted"])  }}
```

On its own, Tauri wires up a pair of commands for the plugin: `checkPermissions` along with `requestPermissions`. Either one is callable straight from JavaScript or Rust:

- JavaScript
- Rust

```
import { invoke, PermissionState } from '@tauri-apps/api/core'
interface Permissions {  postNotification: PermissionState}
// check permission stateconst permission = await invoke<Permissions>('plugin:<plugin-name>|checkPermissions')
if (permission.postNotification === 'prompt-with-rationale') {  // show information to the user about why permission is needed}
// request permissionif (permission.postNotification.startsWith('prompt')) {  const state = await invoke<Permissions>('plugin:<plugin-name>|requestPermissions', { permissions: ['postNotification'] })}
```

```
use serde::{Serialize, Deserialize};use tauri::{plugin::PermissionState, Runtime};
#[derive(Deserialize)]#[serde(rename_all = "camelCase")]struct PermissionResponse {  pub post_notification: PermissionState,}
#[derive(Serialize)]#[serde(rename_all = "camelCase")]struct RequestPermission {  post_notification: bool,}
impl<R: Runtime> Notification<R> {  pub fn request_post_notification_permission(&self) -> crate::Result<PermissionState> {    self.0      .run_mobile_plugin::<PermissionResponse>("requestPermissions", RequestPermission { post_notification: true })      .map(|r| r.post_notification)      .map_err(Into::into)  }
  pub fn check_permissions(&self) -> crate::Result<PermissionResponse> {    self.0      .run_mobile_plugin::<PermissionResponse>("checkPermissions", ())      .map_err(Into::into)  }}
```

### Plugin Events

At any moment a plugin may dispatch events by calling the `trigger` function:

- Android
- iOS

```
@TauriPluginclass ExamplePlugin(private val activity: Activity): Plugin(activity) {    override fun load(webView: WebView) {      trigger("load", JSObject())    }
    override fun onNewIntent(intent: Intent) {      // handle new intent event      if (intent.action == Intent.ACTION_VIEW) {        val data = intent.data.toString()        val event = JSObject()        event.put("data", data)        trigger("newIntent", event)      }    }
    @Command    fun openCamera(invoke: Invoke) {      val payload = JSObject()      payload.put("open", true)      trigger("camera", payload)    }}
```

```
class ExamplePlugin: Plugin {  @objc public override func load(webview: WKWebView) {    trigger("load", data: [:])  }
  @objc public func openCamera(_ invoke: Invoke) {    trigger("camera", data: ["open": true])  }}
```

From the NPM package, you reach those helper functions through the [`addPluginListener`](https://v2.tauri.app/reference/javascript/api/namespacecore/#addpluginlistener) helper:

```
import { addPluginListener, PluginListener } from '@tauri-apps/api/core';
export async function onRequest(  handler: (url: string) => void): Promise<PluginListener> {  return await addPluginListener(    '<plugin-name>',    'event-name',    handler  );}
```

---
