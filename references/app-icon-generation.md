# Generate app icons

[Topic context](app-icons-index.md) / [Previous source section](npm-cargo-dependency-updates.md) / [Next source section](platform-icon-format-requirements.md)

Context: Tauri v2: Generate app icons. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-app-icons"></a> <a id="source-app-icons-1"></a> <a id="source-command-usage"></a>

## App Icons

### App Icons

Out of the box, Tauri bundles a default iconset built around its own logo — clearly NOT something you'd release with your finished application. Tauri addresses this everyday need through the `icon` command, which reads a source file (`"./app-icon.png"` unless told otherwise) and produces every icon the various platforms require.

### Command Usage

- npm
- yarn
- pnpm
- deno
- cargo

```sh
npm run tauri icon
```

```sh
yarn tauri icon
```

```sh
pnpm tauri icon
```

```sh
deno task tauri icon
```

```sh
cargo tauri icon
```

     Terminal window

```console
> pnpm tauri icon --help

Generate various icons for all major platforms

Usage: pnpm run tauri icon [OPTIONS] [INPUT]

Arguments:
  [INPUT]  Path to the source icon (squared PNG or SVG file with transparency) [default: ./app-icon.png]

Options:
  -o, --output <OUTPUT>        Output directory. Default: 'icons' directory next to the tauri.conf.json file
  -v, --verbose...             Enables verbose logging
  -p, --png <PNG>              Custom PNG icon sizes to generate. When set, the default icons are not generated
      --ios-color <IOS_COLOR>  The background color of the iOS icon - string as defined in the W3C's CSS Color Module Level 4 <https://www.w3.org/TR/css-color-4/> [default: #fff]
  -h, --help                   Print help
  -V, --version                Print version
```

By default the **desktop** icons land in your `src-tauri/icons` folder, and from there they're pulled into your compiled app without any extra effort. Should you want them drawn from somewhere else, adjust this portion of the `tauri.conf.json` file:

```json
{
  "bundle": {
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ]
  }
}
```

As for the **mobile** icons, those drop straight into the Xcode and Android Studio projects!

