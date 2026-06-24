# Crabnebula Devtools, Linux Graphics, Neovim, Rustrover

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

## Overview

Topics in this reference: Debug; CrabNebula DevTools; the section on Linux Graphics Issues; plus Debug in Neovim and Debug in JetBrains IDEs.

## Debug

### Debug

Because Tauri stitches together so many components, you will occasionally hit a bug that needs investigation. Error output surfaces in several different spots, and the framework ships a handful of utilities that ease the work of tracking issues down.

### Development Only Code

Sprinkling diagnostic statements throughout your source is among the handiest debugging techniques available. The catch is that such statements rarely belong in a shipped product, so being able to detect at runtime whether the build is a development one becomes valuable.

### In Rust

```
fn main() {  // Whether the current instance was started with `tauri dev` or not.  #[cfg(dev)]  {    // `tauri dev` only code  }  if cfg!(dev) {    // `tauri dev` only code  } else {    // `tauri build` only code  }  let is_dev: bool = tauri::is_dev();
  // Whether debug assertions are enabled or not. This is true for `tauri dev` and `tauri build --debug`.  #[cfg(debug_assertions)]  {    // Debug only code  }  if cfg!(debug_assertions) {    // Debug only code  } else {    // Production only code  }}
```

### Rust Console

Start your error hunt at the Rust Console, which lives in whatever terminal launched a command such as `tauri dev`. To emit text to that console from inside a Rust file, reach for the snippet below:

```
println!("Message from Rust: {}", msg);
```

When a fault hides in your Rust source, the compiler tends to surface plenty of detail about it. Suppose `tauri dev` blows up; you can relaunch it as follows on Linux and macOS:

```
RUST_BACKTRACE=1 tauri dev
```

or, under Windows with PowerShell, this way:

```
$env:RUST_BACKTRACE=1tauri dev
```

Running that produces a fine-grained stack trace. As a rule, the compiler assists you by surfacing thorough diagnostics about whatever went wrong, for instance:

```
error[E0425]: cannot find value `sun` in this scope  --> src/main.rs:11:5   |11 |     sun += i.to_string().parse::<u64>().unwrap();   |     ^^^ help: a local variable with a similar name exists: `sum`
error: aborting due to previous error
For more information about this error, try `rustc --explain E0425`.
```

### WebView Console

Within the WebView, right-click and pick `Inspect Element`. Doing so launches a web-inspector much like the Chrome or Firefox developer tools you already know. The inspector can also be summoned with the `Ctrl + Shift + i` shortcut on Linux and Windows, or `Command + Option + i` over on macOS.

Which inspector appears depends on the platform: the webkit2gtk WebInspector shows up under Linux, Safari’s inspector under macOS, and the Microsoft Edge DevTools under Windows.

### Opening Devtools Programmatically

Toggling whether the inspector window is shown is done through the [`WebviewWindow::open_devtools`](https://docs.rs/tauri/2.0.0/tauri/webview/struct.WebviewWindow.html#method.open_devtools) and [`WebviewWindow::close_devtools`](https://docs.rs/tauri/2.0.0/tauri/webview/struct.WebviewWindow.html#method.close_devtools) functions:

```
tauri::Builder::default()  .setup(|app| {    #[cfg(debug_assertions)] // only include this code on debug builds    {      let window = app.get_webview_window("main").unwrap();      window.open_devtools();      window.close_devtools();    }    Ok(())  });
```

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

```
npm run tauri build -- --debug
```

```
yarn tauri build --debug
```

```
pnpm tauri build --debug
```

```
deno task tauri build --debug
```

```
bun tauri build --debug
```

```
cargo tauri build --debug
```

As with the ordinary build and dev workflows, this command is slow on its initial pass yet noticeably quicker every time after that. Your finished bundle ships with the development console turned on and lands in `src-tauri/target/debug/bundle`.

Launching a compiled app straight from the terminal is another option, and it surfaces the Rust compiler notes (should anything fail) or your `println` output. Navigate to `src-tauri/target/(release|debug)/[app name]`, then either invoke it from your console or double-click the executable in the filesystem (be aware that, with the latter, the console vanishes the moment an error occurs).

##### Enable Devtools Feature

If you want devtools active inside **production builds**, turn on the `devtools` Cargo feature within the `src-tauri/Cargo.toml` file:

```
[dependencies]tauri = { version = "...", features = ["...", "devtools"] }
```

### Debugging the Core Process

Since Rust drives the Core process, GDB or LLDB are both viable for debugging it. The [Debugging in VS Code](https://v2.tauri.app/develop/debug/vscode/) guide walks through applying the LLDB VS Code Extension to step through a Tauri application's Core Process.

---

## CrabNebula DevTools

### CrabNebula DevTools

Through its collaboration with the Tauri project, [CrabNebula](https://crabnebula.dev/) offers a no-cost [DevTools](https://crabnebula.dev/devtools/) application for Tauri. With it you can instrument your Tauri app: it records the bundled assets, the Tauri configuration file, plus logs and spans, then serves a web frontend that renders all of that data live as it happens.

The CrabNebula DevTools let you examine every log event your app emits (dependency logs included), measure how your command invocations perform alongside broader Tauri API activity, and work through a dedicated view for Tauri events and commands that exposes their payload, responses, nested logs, and execution spans.

Switching on the CrabNebula DevTools begins with adding the devtools crate:

```
cargo add tauri-plugin-devtools@2.0.0
```

Then bring the plugin up at the earliest opportunity inside your main function:

```
fn main() {    // This should be called as early in the execution of the app as possible    #[cfg(debug_assertions)] // only enable instrumentation in development builds    let devtools = tauri_plugin_devtools::init();
    let mut builder = tauri::Builder::default();
    #[cfg(debug_assertions)]    {        builder = builder.plugin(devtools);    }
    builder        .run(tauri::generate_context!())        .expect("error while running tauri application");}
```

After that, start your app the usual way; assuming the configuration is right, devtools emits this message:

(Figure: DevTools message on terminal)

Further detail lives in the [CrabNebula DevTools](https://docs.crabnebula.dev/devtools/get-started/) documentation.

---

## Linux Graphics Issues

### Linux Graphics Issues

Under Linux, rendering in Tauri goes through WebKitGTK. With certain configurations, NVIDIA GPUs being the usual culprit, WebKitGTK and the graphics driver fail to agree, producing everything from an empty window to small visual glitches. The known symptoms and their fixes are gathered on this page. The initial reports are documented at [tauri-apps/tauri#9394](https://github.com/tauri-apps/tauri/issues/9394).

### Common symptoms

- A window appears yet remains empty or all white.
- Flickering occurs in the window, particularly during a resize.
- During a resize the app crashes and prints nothing helpful.
- The console reports `AcceleratedSurfaceDMABuf was unable to construct a complete framebuffer`.
- The console reports `Gdk-Message: Error 71 (Protocol error) dispatching to Wayland display.`

The root cause for the majority of these is the WebKitGTK DMABUF renderer asking for buffer formats that the NVIDIA driver simply will not supply. Upstream conversation can be found on the [WebKitGTK bug tracker](https://bugs.webkit.org/show_bug.cgi?id=261874) and in the [NVIDIA forums](https://forums.developer.nvidia.com/t/geforce-rtx-4070-flickering-issue-when-using-the-dmabuf-renderer-in-webkitgtk/274741).

### Workarounds

Work through these in sequence; the ones near the top preserve hardware acceleration.

1. Confirm that kernel mode setting is active. With NVIDIA drivers predating version 545 you frequently have to pass `nvidia_drm.modeset=1` as a kernel parameter.
2. Set `__NV_DISABLE_EXPLICIT_SYNC=1`. More often than not this clears the Wayland `Error 71` crash and carries no performance penalty.
3. Set `WEBKIT_DISABLE_DMABUF_RENDERER=1`. This resolves both the DMABUF framebuffer error and the `Error 71` crash, though you forfeit the quicker rendering route.
4. Set `WEBKIT_DISABLE_COMPOSITING_MODE=1`. Reserve this as the final option for the silent crash during a resize, since it switches off accelerated compositing altogether.

For testing you can export these from your shell, or assign them inside `main()` ahead of the webview's creation so that your users never need to:

```
fn main() {  // Workaround for WebKitGTK on NVIDIA, see tauri-apps/tauri#9394  #[cfg(target_os = "linux")]  std::env::set_var("WEBKIT_DISABLE_DMABUF_RENDERER", "1");
  tauri::Builder::default()    // ...}
```

Reserve an unconditional override of this kind for cases where you have confirmed the bug actually hits your app. Otherwise you strip the faster path from everybody, even those whose setups were fine.

### Silent failures: WebGL and canvas

Some issues neither crash nor raise an error at all. WebGL and canvas output may quietly fall onto a sluggish path even as the rest of your app appears healthy. A pair of factors makes spotting this from within your frontend difficult:

- Creating a WebGL2 context still reports success even where a software rasterizer or a slow presentation route sits underneath it. Nothing throws, so there is no error to catch.
- For fingerprinting protection, WebKitGTK hides the WebGL renderer string. On any Linux box `WEBGL_debug_renderer_info` comes back as `Apple GPU`, leaving you no way to learn what genuinely drives the context.

Day to day, the giveaway is laggy input or poor frame rates within WebGL-intensive screens such as terminal emulators, editors, maps, and charts, despite that identical code running smoothly in an ordinary browser. Should your app rely on a WebGL rendering path, supply a non-WebGL fallback for Linux and think about surfacing a setting users can flip, rather than counting on the context to report the truth.

---

## Debug in Neovim

### Debug in Neovim

Quite a few plugins exist for debugging Rust within Neovim. What follows walks you through configuring `nvim-dap` along with a couple of companion plugins so you can debug a Tauri application.

### Prerequisites

The `nvim-dap` extension depends on the `codelldb` binary. Grab the build that matches your platform from [https://github.com/vadimcn/codelldb/releases](https://github.com/vadimcn/codelldb/releases) and extract the archive. The `nvim-dap` configuration will reference it shortly.

### Configuring nvim-dap

Add the [`nvim-dap`](https://github.com/mfussenegger/nvim-dap) and [`nvim-dap-ui`](https://github.com/rcarriga/nvim-dap-ui) plugins. Either follow what their github pages describe or just lean on whichever plugin manager you prefer. Keep in mind that `nvim-dap-ui` depends on the `nvim-nio` plugin.

After that, wire the plugin into your Neovim configuration:

init.lua

```
local dap = require("dap")
dap.adapters.codelldb = {  type = 'server',  port = "${port}",  executable = {    -- Change this to your path!    command = '/opt/codelldb/adapter/codelldb',    args = {"--port", "${port}"},  }}
dap.configurations.rust= {  {    name = "Launch file",    type = "codelldb",    request = "launch",    program = function()      return vim.fn.input('Path to executable: ', vim.fn.getcwd() .. '/target/debug/', 'file')    end,    cwd = '${workspaceFolder}',    stopOnEntry = false  },}
```

With this arrangement, every time you start the debugger it prompts you to indicate which Tauri App binary you intend to debug.

If you like, configure the `nvim-dap-ui` plugin so the debugger view opens and closes on its own whenever a debugging session begins or ends:

init.lua

```
local dapui = require("dapui")dapui.setup()
dap.listeners.before.attach.dapui_config = function()  dapui.open()enddap.listeners.before.launch.dapui_config = function()  dapui.open()enddap.listeners.before.event_terminated.dapui_config = function()  dapui.close()enddap.listeners.before.event_exited.dapui_config = function()  dapui.close()end
```

Finally, the editor's standard rendering of breakpoints can be customized:

init.lua

```
vim.fn.sign_define('DapBreakpoint',{ text ='🟥', texthl ='', linehl ='', numhl =''})vim.fn.sign_define('DapStopped',{ text ='▶️', texthl ='', linehl ='', numhl =''})
```

### Starting the dev server

Because the Tauri CLI isn't the thing starting our app here, the development server won't come up by itself. The [overseer](https://github.com/stevearc/overseer.nvim/tree/master) plugin lets you drive the development server's state straight from Neovim.

For managing tasks that run in the background, your best bet is a [VS Code style task](https://github.com/stevearc/overseer.nvim/blob/master/doc/guides.md#vs-code-tasks) configuration. Set this up by adding a `.vscode/tasks.json` file within the project directory.

A sample task configuration for a project built around `trunk` appears below.

.vscode/tasks.json

```
{  "version": "2.0.0",  "tasks": [    {      "type": "process",      "label": "dev server",      "command": "trunk",      "args": ["serve"],      "isBackground": true,      "presentation": {        "revealProblems": "onProblem"      },      "problemMatcher": {        "pattern": {          "regexp": "^error:.*",          "file": 1,          "line": 2        },        "background": {          "activeOnStart": false,          "beginsPattern": ".*Rebuilding.*",          "endsPattern": ".*server listening at:.*"        }      }    }  ]}
```

### Example key bindings

Sample key bindings for launching and steering debugging sessions follow.

init.lua

```
vim.keymap.set('n', '<F5>', function() dap.continue() end)vim.keymap.set('n', '<F6>', function() dap.disconnect({ terminateDebuggee = true }) end)vim.keymap.set('n', '<F10>', function() dap.step_over() end)vim.keymap.set('n', '<F11>', function() dap.step_into() end)vim.keymap.set('n', '<F12>', function() dap.step_out() end)vim.keymap.set('n', '<Leader>b', function() dap.toggle_breakpoint() end)vim.keymap.set('n', '<Leader>o', function() overseer.toggle() end)vim.keymap.set('n', '<Leader>R', function() overseer.run_template() end)
```

---

## Debug in JetBrains IDEs

### Debug in JetBrains IDEs

Here we configure JetBrains RustRover to debug the [Core Process of your Tauri app](https://v2.tauri.app/concept/process-model/#the-core-process). For the most part, the same steps carry over to IntelliJ and CLion as well.

### Setting up a Cargo project

Whether the project directory counts as a Cargo project hinges on the frontend stack the project adopts. Tauri's default behavior is to drop the Rust project into a subdirectory named `src-tauri`. Only when Rust also handles the frontend does it set up a Cargo project at the root.

When no `Cargo.toml` exists at the top level, attaching the project by hand becomes necessary. Bring up the Cargo tool window (from the main menu, choose **View | Tool Windows | Cargo**), press **+** (**Attach Cargo Project**) on the toolbar, and pick the `src-tauri/Cargo.toml` file.

Another route is to build a top-level Cargo workspace yourself, placing the file shown below into the project's root directory:

Cargo.toml

```
[workspace]members = ["src-tauri"]
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

```
    "beforeDevCommand": "pnpm dev"
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
