# Choose a state mutex

Open when choosing synchronous or async mutexes for shared state.

[Topic context](managed-state-mutex-selection.md)

### Mutability

Rust won't let you write straight into a value that several threads share, nor one whose ownership runs through a reference-counted pointer like [`Arc`](https://doc.rust-lang.org/stable/std/sync/struct.Arc.html) (or, in Tauri, [`State`](https://docs.rs/tauri/latest/tauri/struct.State.html)). Permitting that would open the door to data races — picture two writes landing at the same moment.

The way past this restriction is a pattern called [interior mutability](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html). One option is to wrap your state in the standard library’s [`Mutex`](https://doc.rust-lang.org/stable/std/sync/struct.Mutex.html), which lets you take a lock before a change and release it once the change is finished.

```rust
use std::sync::Mutex;

use tauri::{Builder, Manager};

#[derive(Default)]
struct AppState {
  counter: u32,
}

fn main() {
  Builder::default()
    .setup(|app| {
      app.manage(Mutex::new(AppState::default()));
      Ok(())
    })
    .run(tauri::generate_context!())
    .unwrap();
}
```

With that in place, locking the mutex grants you write access to the state:

```rust
let state = app.state::<Mutex<AppState>>();

// Lock the mutex to get mutable access:
let mut state = state.lock().unwrap();

// Modify the state:
state.counter += 1;
```

Once execution leaves the scope — or the `MutexGuard` is dropped for any other reason — the lock releases on its own, freeing up the rest of your app to read and change the contained data.

### When to use an async mutex

As the [Tokio documentation](https://docs.rs/tokio/latest/tokio/sync/struct.Mutex.html#which-kind-of-mutex-should-you-use) points out, reaching for the standard library’s [`Mutex`](https://doc.rust-lang.org/stable/std/sync/struct.Mutex.html) rather than an async variant like Tokio’s is frequently a perfectly good choice:

>

Despite what many assume, the plain standard-library Mutex is acceptable — and frequently the better pick — even inside asynchronous code … An async mutex really earns its keep when several tasks need shared mutable access to IO resources, a database connection being the classic example.

Working through that linked page in full is worthwhile so the trade-offs between the two become clear. The one scenario that genuinely calls for an async mutex is keeping the `MutexGuard` alive across await points.

### Do you need `Arc`?

In Rust you'll frequently find [`Arc`](https://doc.rust-lang.org/stable/std/sync/struct.Arc.html) spreading ownership of one value across several threads, typically combined with a [`Mutex`](https://doc.rust-lang.org/stable/std/sync/struct.Mutex.html) as `Arc<Mutex<T>>`. That said, anything you hand to [`State`](https://docs.rs/tauri/latest/tauri/struct.State.html) needs no [`Arc`](https://doc.rust-lang.org/stable/std/sync/struct.Arc.html) of its own, since Tauri handles that wrapping on your behalf.

Should the lifetime constraints on `State` block you from sending your state into a freshly spawned thread, move an `AppHandle` across instead and fetch the state inside, following the “Access state with the Manager trait” section further down. Cloning an `AppHandle` is intentionally inexpensive precisely for situations like this.
