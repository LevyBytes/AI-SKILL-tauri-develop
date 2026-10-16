# Mobile development server

Open when starting mobile development and configuring a reachable frontend server.

[Topic context](mobile-server-device-selection.md)

### Developing Your Mobile Application

The mobile workflow mirrors the desktop one closely, except the command you launch is `tauri android dev` or `tauri ios dev`:

- npm
- yarn
- pnpm
- deno
- bun
- cargo

```sh
npm run tauri [android|ios] dev
```

```sh
yarn tauri [android|ios] dev
```

```sh
pnpm tauri [android|ios] dev
```

```sh
deno task tauri [android|ios] dev
```

```sh
bun tauri [android|ios] dev
```

```sh
cargo tauri [android|ios] dev
```

During the very first invocation, Cargo's package manager could take **several minutes** to retrieve and compile all the crates this requires. Because those results are kept in a cache, every run after that is noticeably quicker, recompiling only what you changed.

#### Development Server

On mobile the dev server behaves much like its desktop counterpart, but targeting a physical iOS device means binding it to a specific address that the Tauri CLI supplies through the `TAURI_DEV_HOST` environment variable. That address is by default a public network address, or alternatively the device's own TUN address — a more secure choice that for now relies on Xcode to reach the device.

Going the TUN-address route requires launching Xcode ahead of the dev command and confirming the device shows up over the network under the Window > Devices and Simulators menu. From there, run `tauri ios dev --force-ip-prompt` and pick the iOS device address (an IPv6 address ending in **::2**).

So that the dev server binds to the right host and the iOS device can reach it, adjust its config to honor the `TAURI_DEV_HOST` value whenever that value is set. Below is a sample Vite configuration:

```js
import { defineConfig } from 'vite';

const host = process.env.TAURI_DEV_HOST;

// https://vitejs.dev/config/
export default defineConfig({
  clearScreen: false,
  server: {
    host: host || false,
    port: 1420,
    strictPort: true,
    hmr: host
      ? {
          protocol: 'ws',
          host,
          port: 1421,
        }
      : undefined,
  },
});
```

Consult the setup guide for your particular framework if you need further detail.
