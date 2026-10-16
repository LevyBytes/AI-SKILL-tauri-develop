# Plugin webview lifecycle

Open when controlling navigation or listening after a plugin webview is ready.

[Topic context](plugin-setup-lifecycle-hooks.md)

### on_navigation

- **When**: the web view is trying to carry out navigation
- **Why**: check that the navigation is valid or keep tabs on URL changes

When you return `false`, the navigation is canceled.

src/lib.rs

```rust
use tauri::{Listener, plugin::Builder};

Builder::new("<plugin-name>")
  .on_navigation(|window, url| {
    println!("window {} is navigating to {}", window.label(), url);
    // Cancels the navigation if forbidden
    url.scheme() != "forbidden"
  })
```

### on_webview_ready

- **When**: a new window has finished being created
- **Why**: run a startup script against each window
  src/lib.rs

```rust
use tauri::{Listener, plugin::Builder};

Builder::new("<plugin-name>")
  .on_webview_ready(|window| {
    window.listen("content-loaded", |event| {
      println!("webview content has been loaded");
    });
  })
```
