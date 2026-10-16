# JetBrains Cargo and development server debugging

[Topic context](debugging-tools-index.md) / [Previous source section](neovim-tauri-debugging.md)

Context: Tauri v2: JetBrains Cargo and development server debugging. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-debug-in-jetbrains-ides"></a> <a id="source-debug-in-jetbrains-ides-1"></a> <a id="source-setting-up-a-cargo-project"></a> <a id="source-setting-up-run-configurations"></a> <a id="source-tauri-app"></a> <a id="source-development-server"></a> <a id="source-launching-a-debugging-session"></a>

## Debug in JetBrains IDEs

### Debug in JetBrains IDEs

Here we configure JetBrains RustRover to debug the [Core Process of your Tauri app](https://v2.tauri.app/concept/process-model/#the-core-process). For the most part, the same steps carry over to IntelliJ and CLion as well.

### Setting up a Cargo project

Whether the project directory counts as a Cargo project hinges on the frontend stack the project adopts. Tauri's default behavior is to drop the Rust project into a subdirectory named `src-tauri`. Only when Rust also handles the frontend does it set up a Cargo project at the root.

When no `Cargo.toml` exists at the top level, attaching the project by hand becomes necessary. Bring up the Cargo tool window (from the main menu, choose **View | Tool Windows | Cargo**), press **+** (**Attach Cargo Project**) on the toolbar, and pick the `src-tauri/Cargo.toml` file.

Another route is to build a top-level Cargo workspace yourself, placing the file shown below into the project's root directory:

Cargo.toml

```toml
[workspace]
members = ["src-tauri"]
```

Confirm the project has finished loading before moving on. Seeing every module and target of the workspace listed in the Cargo tool window means you are ready to continue.

### Setting up Run Configurations

Two distinct Run/Debug configurations have to be created:

- one that starts the Tauri app under debugging mode,
- and a second that runs whichever frontend development server you prefer.

### Tauri App

1. From the main menu, open **Run | Edit Configurations**.
2. Within the **Run/Debug Configurations** dialog:

- Add a fresh configuration by clicking **+** on the toolbar and choosing **Cargo**.

(Figure: Add Run/Debug Configuration)

Once that exists, RustRover must be set up so that it tells Cargo to compile our app with no default features turned on. That instructs Tauri to pull from your development server rather than loading assets off the disk. The Tauri CLI ordinarily supplies this flag, yet because we are bypassing the CLI entirely, the flag has to be added by hand.

(Figure: Add --no-default-features flag)

At this point you may, if you wish, give the Run/Debug Configuration a more memorable label; here we went with “Run Tauri App”, though any name you like works just as well.

(Figure: Rename Configuration)

### Development Server

That earlier configuration leans on Cargo directly to compile the Rust application and hook the debugger onto it. Doing so skips the Tauri CLI outright, which means capabilities such as `beforeDevCommand` and `beforeBuildCommand` will **not** run. The remedy is to launch the development server ourselves.

Building the matching Run configuration first requires confirming which development server is genuinely in play. Open the `src-tauri/tauri.conf.json` file and locate this line:

```json
{ "build": { "beforeDevCommand": "pnpm dev" } }
```

When you are on `npm`, `pnpm`, or `yarn`, the **npm** Run Configuration is one option, such as:

(Figure: NPM Configuration)

Double-check that the **Command**, **Scripts**, and **Package Manager** fields all hold the right values.

Should your development server be `trunk`, as used with Rust-based WebAssembly frontend frameworks, the general-purpose **Shell Script** Run Configuration is a fit:

(Figure: Trunk Serve Configuration)

### Launching a Debugging Session

Kicking off a debugging session means starting your development server first, then beginning to debug the Tauri App by pressing the **Debug** button beside the Run Configurations Switcher. Breakpoints set in any Rust file across your project are picked up automatically by RustRover, which halts at the very first one it reaches.

(Figure: Debug Session)

Going forward, you are free to inspect what your variables hold, advance deeper into the code, and observe the fine detail of what happens at runtime.

---
