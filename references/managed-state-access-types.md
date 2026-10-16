# Access managed state and preserve its exact type

[Topic context](managed-state-index.md) / [Previous source section](managed-state-mutex-selection.md) / [Next source section](npm-cargo-dependency-updates.md)

Context: Tauri v2: Access managed state and preserve its exact type. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-accessing-state"></a> <a id="source-access-state-in-commands"></a> <a id="source-async-commands"></a> <a id="source-managerhttpsdocsrstaurilatesttauritraitmanagerhtml-trait"></a> <a id="source-mismatching-types"></a>

### Accessing State

### Access state in commands

```rust
#[tauri::command]
fn increase_counter(state: State<'_, Mutex<AppState>>) -> u32 {
  let mut state = state.lock().unwrap();
  state.counter += 1;
  state.counter
}
```

To dig deeper into commands, head to [Calling Rust from the Frontend](https://v2.tauri.app/develop/calling-rust/).

#### Async commands

Should your commands be `async` and you'd rather rely on Tokio’s async [`Mutex`](https://docs.rs/tokio/latest/tokio/sync/struct.Mutex.html), the configuration mirrors what you've already seen, and reaching the state looks like this:

```rust
#[tauri::command]
async fn increase_counter(state: State<'_, Mutex<AppState>>) -> Result<u32, ()> {
  let mut state = state.lock().await;
  state.counter += 1;
  Ok(state.counter)
}
```

This borrowed-State async example uses a Result return type to satisfy the command macro. Async commands without borrowed arguments may return other serializable types.

### [`Manager`](https://docs.rs/tauri/latest/tauri/trait.Manager.html) trait

Now and then the state has to be reached from somewhere other than a command — say, a separate thread or an event handler such as `on_window_event`. When that happens, call the `state()` method on any type carrying the [`Manager`](https://docs.rs/tauri/latest/tauri/trait.Manager.html) trait, the `AppHandle` among them, to obtain it:

```rust
use std::sync::Mutex;
use tauri::{Builder, Window, WindowEvent, Manager};

#[derive(Default)]
struct AppState {
  counter: u32,
}

// In an event handler:
fn on_window_event(window: &Window, _event: &WindowEvent) {
    // Get a handle to the app so we can get the global state.
    let app_handle = window.app_handle();
    let state = app_handle.state::<Mutex<AppState>>();

    // Lock the mutex to mutably access the state.
    let mut state = state.lock().unwrap();
    state.counter += 1;
}

fn main() {
  Builder::default()
    .setup(|app| {
      app.manage(Mutex::new(AppState::default()));
      Ok(())
    })
    .on_window_event(on_window_event)
    .run(tauri::generate_context!())
    .unwrap();
}
```

This approach earns its place whenever command injection isn't available to you — for instance, when relocating state into a thread is simpler via an `AppHandle`, or when you're operating outside any command altogether.

### Mismatching Types

When it suits you, defining a type alias around your state guards against this slip-up:

```rust
use std::sync::Mutex;

#[derive(Default)]
struct AppStateInner {
  counter: u32,
}

type AppState = Mutex<AppStateInner>;
```

Just be careful to apply the alias exactly as defined; enclosing it inside another [`Mutex`](https://doc.rust-lang.org/stable/std/sync/struct.Mutex.html) brings the very same problem right back.

---

