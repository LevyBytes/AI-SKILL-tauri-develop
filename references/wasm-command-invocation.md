# Invoke commands from WASM

Open when binding the Tauri invoke function from a Rust WASM frontend.

[Topic context](rust-command-registration-wasm.md)

#### WASM

Should a Rust frontend call `invoke()` with no arguments, your frontend code has to be adjusted as shown below, since optional arguments are something Rust does not provide.

```rust
#[wasm_bindgen]
extern "C" {
    // invoke without arguments
    #[wasm_bindgen(js_namespace = ["window", "__TAURI__", "core"], js_name = invoke)]
    async fn invoke_without_args(cmd: &str) -> JsValue;

    // invoke with arguments (default)
    #[wasm_bindgen(js_namespace = ["window", "__TAURI__", "core"])]
    async fn invoke(cmd: &str, args: JsValue) -> JsValue;

    // They need to have different names!
}
```
