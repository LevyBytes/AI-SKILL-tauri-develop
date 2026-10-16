# Source changes, browser tools and source control

[Topic context](development-configuration-index.md) / [Previous source section](mobile-ide-inspector-troubleshooting.md) / [Next source section](tauri-configuration-formats.md)

Context: Tauri v2: Source changes, browser tools and source control. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-reacting-to-source-code-changes"></a> <a id="source-using-the-browser-devtools"></a> <a id="source-source-control"></a>

### Reacting to Source Code Changes

Just as your webview mirrors edits as they happen, `tauri dev` keeps an eye on the `src-tauri` folder along with its dependent crates across the workspace, so any change there triggers an automatic rebuild and restart of your application.

If you would rather turn this off, attach the `--no-watch` flag to the `tauri dev` command.

When you want particular files left out of the watch, set up `.taurignore` files, which behave just like ordinary `.gitignore` files:

```txt
build/
src/generated/*.rs
deny.toml
```

Typically these `.taurignore` files live in the `src-tauri` directory or at the [cargo workspace](https://doc.rust-lang.org/cargo/reference/workspaces.html) root. As things stand, `tauri dev` searches for `.taurignore` files throughout the shared ancestor of the watched folders and the Cargo workspace root.

### Using the Browser DevTools

Because Tauri's APIs function solely inside your app window, the moment you begin relying on them you can no longer load your frontend in your operating system's browser.

Should you favor your browser's own developer tooling, set up [tauri-invoke-http](https://github.com/tauri-apps/tauri-invoke-http) to route Tauri API calls over an HTTP server.

### Source Control

Within your project repository, you **SHOULD** check `src-tauri/Cargo.lock` into git alongside `src-tauri/Cargo.toml`, since Cargo relies on the lockfile to make builds deterministic. For that reason, every application is advised to track its `Cargo.lock`. By contrast, you **SHOULD NOT** commit the `src-tauri/target` folder or anything it holds.

---

