# Command arguments, return data and array buffers

[Rust commands and events](rust-commands-events-index.md) / [Previous source section](rust-command-registration-wasm.md) / [Next source section](rust-command-error-serialization.md)

Context: Tauri v2: Command arguments, return data and array buffers. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-passing-arguments"></a> <a id="source-returning-data"></a> <a id="source-returning-array-buffers"></a>

### Passing Arguments

Arguments are something your command handlers are able to accept:

```rust
#[tauri::command]
fn my_custom_command(invoke_message: String) {
  println!("I was invoked from JavaScript, with this message: {}", invoke_message);
}
```

Pass the arguments along inside a JSON object whose keys use camelCase:

```javascript
invoke('my_custom_command', { invokeMessage: 'Hello!' });
```

Any type is acceptable for an argument, provided it implements [`serde::Deserialize`](https://docs.serde.rs/serde/trait.Deserialize.html).

### Returning Data

Returning data is also within the reach of command handlers:

```rust
#[tauri::command]
fn my_custom_command() -> String {
  "Hello from Rust!".into()
}
```

A promise comes back from the `invoke` function, and it resolves carrying whatever value was returned:

```javascript
invoke('my_custom_command').then((message) => console.log(message));
```

Data you return may be of any type, so long as it implements [`serde::Serialize`](https://docs.serde.rs/serde/trait.Serialize.html).

#### Returning Array Buffers

Whenever the response heads back to the frontend, return values that implement [`serde::Serialize`](https://docs.serde.rs/serde/trait.Serialize.html) get serialized into JSON. Handing back something bulky — a file, say, or a downloaded HTTP response — through that path can bog your app down. For an efficient way to send array buffers instead, reach for [`tauri::ipc::Response`](https://docs.rs/tauri/2.0.0/tauri/ipc/struct.Response.html):

```rust
use tauri::ipc::Response;
#[tauri::command]
fn read_file() -> Response {
  let data = std::fs::read("/path/to/file").unwrap();
  tauri::ipc::Response::new(data)
}
```

