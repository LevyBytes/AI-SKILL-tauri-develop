# Platform configuration overrides

Open when merging platform-specific configuration with the base Tauri configuration.

[Topic context](platform-configuration-merging.md)

### Platform-specific Configuration

Beyond the default configuration file, Tauri is able to load a platform-specific configuration drawn from:

- `tauri.linux.conf.json` or `Tauri.linux.toml` for Linux
- `tauri.windows.conf.json` or `Tauri.windows.toml` for Windows
- `tauri.macos.conf.json` or `Tauri.macos.toml` for macOS
- `tauri.android.conf.json` or `Tauri.android.toml` for Android
- `tauri.ios.conf.json` or `Tauri.ios.toml` for iOS

Following the [JSON Merge Patch (RFC 7396)](https://datatracker.ietf.org/doc/html/rfc7396) specification, the platform-specific configuration file is merged into the main configuration object.

As an illustration, suppose you start from this base `tauri.conf.json`:

tauri.conf.json

```json
{
  "productName": "MyApp",
  "bundle": {
    "resources": [
      "./resources"
    ]
  },
  "plugins": {
    "deep-link": {}
  }
}
```

Together with this `tauri.linux.conf.json`:

tauri.linux.conf.json

```json
{
  "productName": "my-app",
  "bundle": {
    "resources": ["./linux-assets"]
  },
  "plugins": {
    "cli": {
      "description": "My app",
      "subcommands": {
        "update": {}
      }
    },
    "deep-link": {}
  }
}
```

The configuration that Linux ends up resolving to is this object:

```json
{
  "productName": "my-app",
  "bundle": {
    "resources": ["./linux-assets"]
  },
  "plugins": {
    "cli": {
      "description": "My app",
      "subcommands": {
        "update": {}
      }
    },
    "deep-link": {}
  }
}
```

Objects merge by key; arrays replace the entire base array. See [build variants](configuration-build-variants.md) for CLI configuration overrides.
