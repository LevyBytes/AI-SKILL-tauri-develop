# Tauri configuration formats

[Topic context](development-configuration-index.md) / [Previous source section](development-watch-browser-git.md) / [Next source section](platform-configuration-merging.md)

Context: Tauri v2: Tauri configuration formats. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-configuration-files-1"></a> <a id="source-configuration-files-2"></a> <a id="source-tauri-config"></a> <a id="source-supported-formats"></a>

## Configuration Files

### Configuration Files

Because Tauri serves as a toolkit for assembling applications, a project's settings can be spread across several files. The ones you will most often encounter are `tauri.conf.json`, `package.json` and `Cargo.toml`. This page gives a short rundown of each so you know which file to reach for.

### Tauri Config

Through the Tauri configuration you specify where your Web app comes from, supply your application's metadata, set up bundles, declare plugin settings, and shape runtime behavior by defining windows, tray icons, menus and so on.

Both the Tauri runtime and the Tauri CLI consume this file. It lets you lay out build settings (for instance the [command run before `tauri build`](https://v2.tauri.app/reference/config/#beforebuildcommand) or [`tauri dev`](https://v2.tauri.app/reference/config/#beforedevcommand-1) kicks in), assign the [name](https://v2.tauri.app/reference/config/#productname) and [version of your app](https://v2.tauri.app/reference/config/#version), [control the Tauri runtime](https://v2.tauri.app/reference/config/#appconfig), and [configure plugins](https://v2.tauri.app/reference/config/#plugins).

### Supported Formats

JSON is what Tauri uses for its config by default. To switch to JSON5 or TOML, add the matching `config-json5` or `config-toml` feature flag onto the `tauri` and `tauri-build` dependencies inside `Cargo.toml`.

Cargo.toml

```toml
[build-dependencies]
tauri-build = { version = "2.0.0", features = [ "config-json5" ] }

[dependencies]
tauri = { version = "2.0.0", features = [  "config-json5" ] }
```

While the layout and the values stay identical no matter the format, the syntax you write must follow the conventions of each particular file's format:

tauri.conf.json5

```json5
{
  build: {
    devUrl: 'http://localhost:3000',
    // start the dev server
    beforeDevCommand: 'npm run dev',
  },
  bundle: {
    active: true,
    icon: ['icons/app.png'],
  },
  app: {
    windows: [
      {
        title: 'MyApp',
      },
    ],
  },
  plugins: {
    updater: {
      pubkey: 'updater pub key',
      endpoints: ['https://my.app.updater/{{target}}/{{current_version}}'],
    },
  },
}
```

Tauri.toml

```toml
[build]
dev-url = "http://localhost:3000"
# start the dev server
before-dev-command = "npm run dev"

[bundle]
active = true
icon = ["icons/app.png"]

[[app.windows]]
title = "MyApp"

[plugins.updater]
pubkey = "updater pub key"
endpoints = ["https://my.app.updater/{{target}}/{{current_version}}"]
```

Bear in mind that comments are allowed in JSON5 and TOML, and TOML lets you write config names in the more idiomatic kebab-case. Across all 3 formats, field names remain case-sensitive.

