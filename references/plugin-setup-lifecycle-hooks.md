# Plugin setup and lifecycle events

[Topic context](plugin-development-index.md) / [Previous source section](plugin-initialization-naming-configuration.md) / [Next source section](plugin-rust-command-apis.md)

Context: Tauri v2: Plugin setup and lifecycle events. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-lifecycle-events"></a> <a id="source-setup"></a> <a id="source-on_navigation"></a> <a id="source-on_webview_ready"></a> <a id="source-on_event"></a> <a id="source-on_drop"></a>

- [Plugin setup and background work](plugin-setup-background-work.md): Open when initializing managed state and background work in plugin setup.
- [Plugin webview lifecycle](plugin-webview-lifecycle.md): Open when controlling navigation or listening after a plugin webview is ready.
- [Plugin event and exit lifecycle](plugin-event-exit-lifecycle.md): Open when handling event-loop notifications, saving state at exit or running plugin teardown.
