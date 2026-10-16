# Debug builds, production inspector and core debugger

[Topic context](debugging-tools-index.md) / [Previous source section](rust-webview-development-diagnostics.md) / [Next source section](crabnebula-devtools-setup.md)

Context: Tauri v2: Debug builds, production inspector and core debugger. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-using-the-inspector-in-production"></a> <a id="source-create-a-debug-build"></a> <a id="source-enable-devtools-feature"></a> <a id="source-debugging-the-core-process"></a>

### Using the Inspector in Production

Out of the box the inspector is available solely in development and debug builds, unless you switch it on through a Cargo feature.

#### Create a Debug Build

Producing a debug build is a matter of invoking the `tauri build --debug` command.

- npm
- yarn
- pnpm
- deno
- bun
- cargo

```sh
npm run tauri build -- --debug
```

```sh
yarn tauri build --debug
```

```sh
pnpm tauri build --debug
```

```sh
deno task tauri build --debug
```

```sh
bun tauri build --debug
```

```sh
cargo tauri build --debug
```

As with the ordinary build and dev workflows, this command is slow on its initial pass yet noticeably quicker every time after that. Your finished bundle ships with the development console turned on and lands in `src-tauri/target/debug/bundle`.

Launching a compiled app straight from the terminal is another option, and it surfaces the Rust compiler notes (should anything fail) or your `println` output. Navigate to `src-tauri/target/(release|debug)/[app name]`, then either invoke it from your console or double-click the executable in the filesystem (be aware that, with the latter, the console vanishes the moment an error occurs).

##### Enable Devtools Feature

If you want devtools active inside **production builds**, turn on the `devtools` Cargo feature within the `src-tauri/Cargo.toml` file:

```toml
[dependencies]
tauri = { version = "...", features = ["...", "devtools"] }
```

### Debugging the Core Process

Since Rust drives the Core process, GDB or LLDB are both viable for debugging it. The [Debugging in VS Code](https://v2.tauri.app/develop/debug/vscode/) guide walks through applying the LLDB VS Code Extension to step through a Tauri application's Core Process.

---

