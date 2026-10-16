# Manual Android and iOS icon requirements

[Topic context](app-icons-index.md) / [Previous source section](app-icon-generation.md)

Context: Tauri v2: Manual Android and iOS icon requirements. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-creating-icons-manually"></a> <a id="source-android"></a> <a id="source-ios"></a>

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
