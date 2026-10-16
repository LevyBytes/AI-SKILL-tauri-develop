# Async commands and JavaScript invocation

[Rust commands and events](rust-commands-events-index.md) / [Previous source section](rust-command-error-serialization.md) / [Next source section](rust-command-context-streaming.md)

Context: Tauri v2: Async commands and JavaScript invocation. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-async-commands"></a> <a id="source-invoking-from-javascript"></a>

### Async Commands

For demanding work, Tauri favors asynchronous commands, since they handle it without locking up or slowing the UI.

**If your command needs to run asynchronously, simply declare it as `async`.**

Borrowed types call for a few extra tweaks. You have two principal paths to choose from:

**Option 1**: Swap the borrowed type for a comparable non-borrowed one — say, `&str` for `String`. Certain types, such as `State<'_, Data>`, won't cooperate with this.

_Example:_

```rust
// Declare the async function using String instead of &str, as &str is borrowed and thus unsupported
#[tauri::command]
async fn my_custom_command(value: String) -> String {
  // Call another async function and wait for it to finish
  some_async_function().await;
  value
}
```

**Option 2**: Enclose the return type within a [`Result`](https://doc.rust-lang.org/std/result/index.html). Implementing this takes a touch more effort, but it covers every type.

Adopt the return type `Result<a, b>`: put in place of `a` whatever type you want back — or `()` when you mean to return `null` — and in place of `b` an error type for when things go awry, or `()` if you want no optional error coming back. As examples:

- `Result<String, ()>` to hand back a String with no error.
- `Result<(), ()>` to give back `null`.
- `Result<bool, Error>` to deliver a boolean, or an error along the lines shown in the Error Handling section earlier.

_Example:_

```rust
// Return a Result<String, ()> to bypass the borrowing issue
#[tauri::command]
async fn my_custom_command(value: &str) -> Result<String, ()> {
  // Call another async function and wait for it to finish
  some_async_function().await;
  // Note that the return value must be wrapped in `Ok()` now.
  Ok(format!(value))
}
```

##### Invoking from JavaScript

Given that calling the command from JavaScript hands back a promise already, it behaves no differently from any other command:

```javascript
invoke('my_custom_command', { value: 'Hello, Async!' }).then(() =>
  console.log('Completed!')
);
```

