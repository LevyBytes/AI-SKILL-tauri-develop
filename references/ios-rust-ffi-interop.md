# Call Rust through iOS FFI

Open when calling Rust functions from Swift and releasing returned memory.

[Topic context](mobile-rust-interop-alignment.md)

### iOS

Since iOS relies purely on plain C FFI, no extra dependencies come into play. Place the hook inside your Swift code together with whatever teardown is needed. You may give these functions any valid name, provided each one carries the `@_silgen_name(FFI_FUNC)` annotation, in which FFI_FUNC stands for the exported Rust function name that Swift will call:

Place these static methods inside the same Swift type. The symbol names identify the Rust exports called by Swift.

```swift
@_silgen_name("hello_world_ffi")
private static func helloWorldFFI(_ name: UnsafePointer<CChar>) -> UnsafeMutablePointer<CChar>?

@_silgen_name("free_hello_result_ffi")
private static func freeHelloResult(_ result: UnsafeMutablePointer<CChar>)

static func helloWorld(name: String) -> String? {
  // Call Rust FFI
  guard let resultPtr = name.withCString({ helloWorldFFI($0) }) else { return nil }

  // Convert C string to Swift String
  let result = String(cString: resultPtr)

  // Free the C string
  freeHelloResult(resultPtr)

  return result
}
```

Now build out the Rust half. Be sure the `extern` functions here line up with the `@_silgen_name` annotations declared over on the Swift side:

```rust
use std::ffi::{c_char, CStr, CString};

#[no_mangle]
pub unsafe extern "C" fn hello_world_ffi(c_name: *const c_char) -> *mut c_char {
    if c_name.is_null() { return std::ptr::null_mut(); }
    let name = match unsafe { CStr::from_ptr(c_name) }.to_str() {
        Ok(s) => s,
        Err(e) => {
            log::error!("[iOS FFI] Failed to convert C string: {}", e);
            return std::ptr::null_mut();
        }
    };

    let result = format!("Hello, {}!", name);

    match CString::new(result) {
        Ok(c_str) => c_str.into_raw(),
        Err(e) => {
            log::error!("[iOS FFI] Failed to create C string: {}", e);
            std::ptr::null_mut()
        }
    }
}

#[no_mangle]
pub unsafe extern "C" fn free_hello_result_ffi(result: *mut c_char) {
    if !result.is_null() {
        drop(unsafe { CString::from_raw(result) });
    }
}
```
