# Rust development console diagnostics

Open when using development-only code, console messages or Rust backtraces.

[Topic context](rust-webview-development-diagnostics.md)

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

```rs
fn main() {
  // Whether the current instance was started with `tauri dev` or not.
  #[cfg(dev)]
  {
    // `tauri dev` only code
  }
  if cfg!(dev) {
    // `tauri dev` only code
  } else {
    // `tauri build` only code
  }
  let is_dev: bool = tauri::is_dev();

  // Whether debug assertions are enabled or not. This is true for `tauri dev` and `tauri build --debug`.
  #[cfg(debug_assertions)]
  {
    // Debug only code
  }
  if cfg!(debug_assertions) {
    // Debug only code
  } else {
    // Production only code
  }
}
```

### Rust Console

Start your error hunt at the Rust Console, which lives in whatever terminal launched a command such as `tauri dev`. To emit text to that console from inside a Rust file, reach for the snippet below:

```rust
println!("Message from Rust: {}", msg);
```

When a fault hides in your Rust source, the compiler tends to surface plenty of detail about it. Suppose `tauri dev` blows up; you can relaunch it as follows on Linux and macOS:

```shell
RUST_BACKTRACE=1 tauri dev
```

or, under Windows with PowerShell, this way:

```powershell
$env:RUST_BACKTRACE=1
tauri dev
```

Running that produces a fine-grained stack trace. As a rule, the compiler assists you by surfacing thorough diagnostics about whatever went wrong, for instance:

```text
error[E0425]: cannot find value `sun` in this scope
  --> src/main.rs:11:5
   |
11 |     sun += i.to_string().parse::<u64>().unwrap();
   |     ^^^ help: a local variable with a similar name exists: `sum`

error: aborting due to previous error

For more information about this error, try `rustc --explain E0425`.
```
