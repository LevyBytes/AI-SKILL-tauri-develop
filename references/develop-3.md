# State Management, Updating Dependencies, Icons

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: State Management; Updating Dependencies; App Icons.

## State Management

### State Management

Most Tauri apps reach a point where they must remember some current condition or oversee the lifecycle of resources tied to the app. With the [`Manager`](https://docs.rs/tauri/latest/tauri/trait.Manager.html) API, Tauri gives you a straightforward path to hold that data and pull it back out whenever a command runs.

A minimal illustration follows:

```
use tauri::{Builder, Manager};
struct AppData {  welcome_message: &'static str,}
fn main() {  Builder::default()    .setup(|app| {      app.manage(AppData {        welcome_message: "Welcome to Tauri!",      });      Ok(())    })    .run(tauri::generate_context!())    .unwrap();}
```

Afterward, retrieving that stored value is possible from anything carrying the [`Manager`](https://docs.rs/tauri/latest/tauri/trait.Manager.html) trait — the [`App`](https://docs.rs/tauri/latest/tauri/struct.App.html) instance being one such example:

```
let data = app.state::<AppData>();
```

The Accessing State section below has additional details, such as how to reach state from within commands.

### Mutability

Rust won't let you write straight into a value that several threads share, nor one whose ownership runs through a reference-counted pointer like [`Arc`](https://doc.rust-lang.org/stable/std/sync/struct.Arc.html) (or, in Tauri, [`State`](https://docs.rs/tauri/latest/tauri/struct.State.html)). Permitting that would open the door to data races — picture two writes landing at the same moment.

The way past this restriction is a pattern called [interior mutability](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html). One option is to wrap your state in the standard library’s [`Mutex`](https://doc.rust-lang.org/stable/std/sync/struct.Mutex.html), which lets you take a lock before a change and release it once the change is finished.

```
use std::sync::Mutex;
use tauri::{Builder, Manager};
#[derive(Default)]struct AppState {  counter: u32,}
fn main() {  Builder::default()    .setup(|app| {      app.manage(Mutex::new(AppState::default()));      Ok(())    })    .run(tauri::generate_context!())    .unwrap();}
```

With that in place, locking the mutex grants you write access to the state:

```
let state = app.state::<Mutex<AppState>>();
// Lock the mutex to get mutable access:let mut state = state.lock().unwrap();
// Modify the state:state.counter += 1;
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

### Accessing State

### Access state in commands

```
#[tauri::command]fn increase_counter(state: State<'_, Mutex<AppState>>) -> u32 {  let mut state = state.lock().unwrap();  state.counter += 1;  state.counter}
```

To dig deeper into commands, head to [Calling Rust from the Frontend](https://v2.tauri.app/develop/calling-rust/).

#### Async commands

Should your commands be `async` and you'd rather rely on Tokio’s async [`Mutex`](https://docs.rs/tokio/latest/tokio/sync/struct.Mutex.html), the configuration mirrors what you've already seen, and reaching the state looks like this:

```
#[tauri::command]async fn increase_counter(state: State<'_, Mutex<AppState>>) -> Result<u32, ()> {  let mut state = state.lock().await;  state.counter += 1;  Ok(state.counter)}
```

Keep in mind that asynchronous commands oblige you to declare a [`Result`](https://doc.rust-lang.org/stable/std/result/index.html) return type.

### [`Manager`](https://docs.rs/tauri/latest/tauri/trait.Manager.html) trait

Now and then the state has to be reached from somewhere other than a command — say, a separate thread or an event handler such as `on_window_event`. When that happens, call the `state()` method on any type carrying the [`Manager`](https://docs.rs/tauri/latest/tauri/trait.Manager.html) trait, the `AppHandle` among them, to obtain it:

```
use std::sync::Mutex;use tauri::{Builder, Window, WindowEvent, Manager};
#[derive(Default)]struct AppState {  counter: u32,}
// In an event handler:fn on_window_event(window: &Window, _event: &WindowEvent) {    // Get a handle to the app so we can get the global state.    let app_handle = window.app_handle();    let state = app_handle.state::<Mutex<AppState>>();
    // Lock the mutex to mutably access the state.    let mut state = state.lock().unwrap();    state.counter += 1;}
fn main() {  Builder::default()    .setup(|app| {      app.manage(Mutex::new(AppState::default()));      Ok(())    })    .on_window_event(on_window_event)    .run(tauri::generate_context!())    .unwrap();}
```

This approach earns its place whenever command injection isn't available to you — for instance, when relocating state into a thread is simpler via an `AppHandle`, or when you're operating outside any command altogether.

### Mismatching Types

When it suits you, defining a type alias around your state guards against this slip-up:

```
use std::sync::Mutex;
#[derive(Default)]struct AppStateInner {  counter: u32,}
type AppState = Mutex<AppStateInner>;
```

Just be careful to apply the alias exactly as defined; enclosing it inside another [`Mutex`](https://doc.rust-lang.org/stable/std/sync/struct.Mutex.html) brings the very same problem right back.

---

## Updating Dependencies

### Updating Dependencies

### Update npm Packages

For projects that depend on the `tauri` package:

- npm
- yarn
- pnpm

```
npm install @tauri-apps/cli@latest @tauri-apps/api@latest
```

```
yarn up @tauri-apps/cli @tauri-apps/api
```

```
pnpm update @tauri-apps/cli @tauri-apps/api --latest
```

From a terminal, you can likewise discover which Tauri release is currently newest by running:

- npm
- yarn
- pnpm

```
npm outdated @tauri-apps/cli
```

```
yarn outdated @tauri-apps/cli
```

```
pnpm outdated @tauri-apps/cli
```

### Update Cargo Packages

To find packages that have fallen behind, lean on [`cargo outdated`](https://github.com/kbknapp/cargo-outdated), or consult the crates.io listings directly: [tauri](https://crates.io/crates/tauri/versions) / [tauri-build](https://crates.io/crates/tauri-build/versions).

Open `src-tauri/Cargo.toml` and set `tauri` plus `tauri-build` to

```
[build-dependencies]tauri-build = "%version%"
[dependencies]tauri = { version = "%version%" }
```

in which `%version%` stands for the matching version number sourced above.

After that, run:

Terminal window

```
cd src-tauricargo update
```

As another route, the `cargo upgrade` command that ships with [cargo-edit](https://github.com/killercup/cargo-edit) performs every one of these steps for you.

### Sync npm Packages and Cargo Crates versions

Because those JavaScript APIs sit on top of Rust code running in the backend, shipping a new capability means raising both halves in step so they stay compatible. Take care that the minor version of the npm package `@tauri-apps/api` lines up with the cargo crate `tauri`

Plugins behave a little differently: changes of this kind can arrive in patch releases, so the npm package and cargo crate versions advance in lockstep and their exact values must remain identical — to give one case, the npm package `@tauri-apps/plugin-fs` and the cargo crate `tauri-plugin-fs` should both sit at the same version (say `2.2.1`)

---

## App Icons

### App Icons

Out of the box, Tauri bundles a default iconset built around its own logo — clearly NOT something you'd release with your finished application. Tauri addresses this everyday need through the `icon` command, which reads a source file (`"./app-icon.png"` unless told otherwise) and produces every icon the various platforms require.

### Command Usage

- npm
- yarn
- pnpm
- deno
- cargo

```
npm run tauri icon
```

```
yarn tauri icon
```

```
pnpm tauri icon
```

```
deno task tauri icon
```

```
cargo tauri icon
```

     Terminal window

```
> pnpm tauri icon --help
Generate various icons for all major platforms
Usage: pnpm run tauri icon [OPTIONS] [INPUT]
Arguments:  [INPUT]  Path to the source icon (squared PNG or SVG file with transparency) [default: ./app-icon.png]
Options:  -o, --output <OUTPUT>        Output directory. Default: 'icons' directory next to the tauri.conf.json file  -v, --verbose...             Enables verbose logging  -p, --png <PNG>              Custom PNG icon sizes to generate. When set, the default icons are not generated      --ios-color <IOS_COLOR>  The background color of the iOS icon - string as defined in the W3C's CSS Color Module Level 4 <https://www.w3.org/TR/css-color-4/> [default: #fff]  -h, --help                   Print help  -V, --version                Print version
```

By default the **desktop** icons land in your `src-tauri/icons` folder, and from there they're pulled into your compiled app without any extra effort. Should you want them drawn from somewhere else, adjust this portion of the `tauri.conf.json` file:

```
{  "bundle": {    "icon": [      "icons/32x32.png",      "icons/128x128.png",      "icons/128x128@2x.png",      "icons/icon.icns",      "icons/icon.ico"    ]  }}
```

As for the **mobile** icons, those drop straight into the Xcode and Android Studio projects!

### Creating icons manually

Choosing to craft these icons on your own — perhaps to give the small sizes a cleaner design, or to avoid leaning on the CLI’s built-in image resizing — means your icons have to satisfy a handful of requirements:

- `icon.icns`: Which layer sizes and names the [`icns`](https://en.wikipedia.org/wiki/Apple_Icon_Image_format) file needs is laid out [in the Tauri repo](https://github.com/tauri-apps/tauri/blob/1.x/tooling/cli/src/helpers/icns.json)
- `icon.ico`: Inside the [`ico`](<https://en.wikipedia.org/wiki/ICO_(file_format)>) file you must supply layers at 16, 24, 32, 48, 64 and 256 pixels. So the ICO renders well _in development_, list the 32px layer ahead of the rest.
- `png`: Each png icon has to be square (width == height), use RGBA (RGB + Transparency), and carry 32bit per pixel (8bit per channel). On desktop the sizes typically asked for are 32, 128, 256, and 512 pixels. At a minimum, aim to cover what `tauri icon` emits: `32x32.png`, `128x128.png`, `128x128@2x.png`, and `icon.png`.

### Android

For Android the png icons follow those same rules but come in a different set of sizes, and they too go straight into the Android Studio project:

- `src-tauri/gen/android/app/src/main/res/`
  - `mipmap-hdpi/`
    - `ic_launcher.png` & `ic_launcher_round.png`: 49x49px
    - `ic_launcher_foreground.png`: 162x162px

  - `mipmap-mdpi/`
    - `ic_launcher.png` & `ic_launcher_round.png`: 48x48px
    - `ic_launcher_foreground.png`: 108x108px

  - `mipmap-xhdpi/`
    - `ic_launcher.png` & `ic_launcher_round.png`: 96x96px
    - `ic_launcher_foreground.png`: 216x216px

  - `mipmap-xxhdpi/`
    - `ic_launcher.png` & `ic_launcher_round.png`: 144x144px
    - `ic_launcher_foreground.png`: 324x324px

  - `mipmap-xxxhdpi/`
    - `ic_launcher.png` & `ic_launcher_round.png`: 192x192px
    - `ic_launcher_foreground.png`: 432x432px

When `tauri icon` isn't an option for you, a solid fallback is Android Studio’s [Image Asset Studio](https://developer.android.com/studio/write/create-app-icons).

### iOS

iOS likewise wants png icons matching those rules, except here they must be **without transparency** and again span their own sizes. These belong directly in the Xcode project at `src-tauri/gen/apple/Assets.xcassets/AppIcon.appiconset/`, and the set looked for is:

- 20px at 1x, 2x, and 3x, plus one extra icon
- 29px at 1x, 2x, and 3x, plus one extra icon
- 40px at 1x, 2x, and 3x, plus one extra icon
- 60px at 2x and 3x
- 76px at 1x and 2x
- 83.5px at 2x
- 512px at 2x, stored as `AppIcon-512@2x.png`

Every file name follows the pattern `AppIcon-{size}x{size}@{scaling}{extra}.png`. Take the 20px case: you'll produce icons measuring 20x20, 40x40 and 60x60, labelled `AppIcon-20x20@1x.png`, `AppIcon-20x20@2x.png`, `AppIcon-20x20@3x.png`, with the `2x` one also kept a second time as `AppIcon-20x20@2x-1.png` (the “extra icon”).

---
