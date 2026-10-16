# Set up a mobile plugin

Open when creating a plugin project and registering its Android and iOS implementations.

[Topic context](mobile-plugin-initialization-configuration.md)

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

```rust
use tauri::Runtime;

impl<R: Runtime> <plugin-name><R> {
  pub fn do_something(&self) {
    // do something that is a shared implementation between desktop and mobile
  }
}
```

Doing it this way makes it easy to expose one API that both commands and plain Rust code can reach for.

### Develop an Android Plugin

On Android a Tauri plugin takes the form of a Kotlin class subclassing `app.tauri.plugin.Plugin` and carrying the `app.tauri.annotation.TauriPlugin` annotation. Any method tagged with `app.tauri.annotation.Command` becomes reachable from Rust or JavaScript.

Although Tauri defaults to Kotlin for the Android side, Java remains an option when that suits you better. Once the plugin exists, right-click its Kotlin plugin class within Android Studio and pick “Convert Kotlin file to Java file” from the menu; Android Studio then steps you through converting the project to Java.

### Develop an iOS Plugin

For iOS, a Tauri plugin is written as a Swift class that derives from the `Plugin` class supplied by the `Tauri` package. Whenever a function bears the `@objc` attribute together with the `(_ invoke: Invoke)` parameter (say `@objc private func download(_ invoke: Invoke) { }`), Rust or JavaScript can invoke it.

Because the plugin is structured as a [Swift package](https://www.swift.org/package-manager/), its package manager handles dependency management for you.

The angle-bracket plugin name in the Rust template stands for your generated plugin wrapper type; substitute that type before compiling.
