# Select a mobile device

Open when choosing a connected device or named simulator for development.

[Topic context](mobile-server-device-selection.md)

#### Device Selection

Out of the box, the mobile dev command attempts to launch your app on whatever device is plugged in, and if none is found it asks you to pick a simulator. Should you wish to set the run target ahead of time, pass the device or simulator name as an argument:

- npm
- yarn
- pnpm
- deno
- bun
- cargo

```sh
npm run tauri ios dev 'iPhone 15'
```

```sh
yarn tauri ios dev 'iPhone 15'
```

```sh
pnpm tauri ios dev 'iPhone 15'
```

```sh
deno task tauri ios dev 'iPhone 15'
```

```sh
bun tauri ios dev 'iPhone 15'
```

```sh
cargo tauri ios dev 'iPhone 15'
```
