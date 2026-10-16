# Call Rust through Android JNI

Open when calling Rust functions from an Android plugin through JNI.

[Topic context](mobile-rust-interop-alignment.md)

### Calling Rust From Mobile Plugins

Writing plugin logic in Rust is frequently the better choice, both for speed and for reuse. Tauri offers no built-in path to reach Rust from inside your plugin code, yet JNI on Android and FFI on iOS let plugins invoke shared code even while the app's WebView sits suspended.

### Android

Inside the `Cargo.toml` belonging to your plugin, list the jni crate among the dependencies:

```toml
[target.'cfg(target_os = "android")'.dependencies]
jni = "0.21"
```

Pull in the application library statically and declare the native functions within your Kotlin code. Here the Kotlin class happens to be `com.example.HelloWorld`, and the Rust side has to cite that complete package name.

Place the following members inside the `com.example.HelloWorld` class; define `TAG` in its companion object and import `android.util.Log`.

```kotlin

init {
  try {
    // Load the native library (libapp_lib.so)
    // This is the shared library built by Cargo with crate-type = ["cdylib"]
    System.loadLibrary("app_lib")
    Log.d(TAG, "Successfully loaded libapp_lib.so")
  } catch (e: UnsatisfiedLinkError) {
    Log.e(TAG, "Failed to load libapp_lib.so", e)
    throw e
  }
}

external fun helloWorld(name: String): String?
```

Next, over in your plugin's Rust code, declare the function that JNI expects to find. Names follow the `Java_package_class_method` shape, so for the class shown earlier it works out to `Java_com_example_HelloWorld_helloWorld`, which is what our `helloWorld` method ends up calling:

```rust
#[cfg(target_os = "android")]
#[no_mangle]
pub extern "system" fn Java_com_example_HelloWorld_helloWorld(
    mut env: jni::JNIEnv,
    _object: jni::objects::JObject,
    name: jni::objects::JString,
) -> jni::sys::jstring {
    let name: String = match env.get_string(&name) {
        Ok(value) => value.into(),
        Err(error) => {
            log::error!("Failed to read JString: {}", error);
            return std::ptr::null_mut();
        }
    };
    match env.new_string(format!("Hello, {}!", name)) {
        Ok(value) => value.into_raw(),
        Err(error) => {
            log::error!("Failed to create JString: {}", error);
            std::ptr::null_mut()
        }
    }
}
```
