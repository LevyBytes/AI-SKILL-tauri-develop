# Mobile IDEs, Web Inspector and troubleshooting

[Topic context](development-configuration-index.md) / [Previous source section](mobile-server-device-selection.md) / [Next source section](development-watch-browser-git.md)

Context: Tauri v2: Mobile IDEs, Web Inspector and troubleshooting. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-using-xcode-or-android-studio"></a> <a id="source-opening-the-web-inspector-1"></a> <a id="source-troubleshooting"></a>

#### Using Xcode or Android Studio

Another option is to drive development through Xcode or Android Studio directly. Leaning on the IDE rather than the command line can make certain development problems easier to diagnose. When you want the mobile IDE to open instead of launching on an attached device or simulator, add the `--open` flag:

- npm
- yarn
- pnpm
- deno
- bun
- cargo

```sh
npm run tauri [android|ios] dev --open
```

```sh
yarn tauri [android|ios] dev --open
```

```sh
pnpm tauri [android|ios] dev --open
```

```sh
deno task tauri [android|ios] dev --open
```

```sh
bun tauri [android|ios] dev --open
```

```sh
cargo tauri [android|ios] dev --open
```

#### Opening the Web Inspector

-

iOS

Reaching the Web Inspector for an iOS app is only possible through Safari.

Launch Safari on your Mac, pick **Safari > Settings** from the menu bar, open **Advanced**, and tick **Show features for web developers**.

When the target is a physical device, you also have to switch on **Web Inspector** under **Settings > Safari > Advanced**.

Once every step is done, a **Develop** menu appears in Safari listing the attached devices and apps available for inspection. Choose your device or simulator, then click **localhost** to bring up the Safari Developer Tools window.

-

Android

Android emulators come with the inspector turned on already, yet physical hardware needs it switched on manually. Plug your Android device into the computer, launch the **Settings** app there, tap **About**, find Build Number, and tap it 7 times. Doing so unlocks Developer Mode along with the **Developer Options** settings on the device.

Next, to permit app debugging, head into the **Developer Options** settings, flip the developer options switch on, and turn on **USB Debugging**.

Android's Web Inspector runs on Google Chrome's DevTools, which you reach by visiting `chrome://inspect` in Chrome on your machine. While your Android app is live, the device or emulator shows up in the remote devices list, and clicking **inspect** beside the matching entry launches the developer tools.

#### Troubleshooting

1. Error running build script on Xcode

Tauri ties into the iOS Xcode project by adding a build phase that runs the Tauri CLI to compile the Rust source into a library loaded at runtime. Since that build phase runs within the Xcode process context, it may not pick up shell tweaks like PATH additions, so take care with utilities such as Node.js version managers that might not work in that environment.

1. Network permission prompt on first iOS app execution

The first run of `tauri ios dev` may trigger an iOS prompt asking permission to discover and connect to devices on your local network. That prompt appears because reaching your dev server from an iOS device requires the server to be reachable across the local network. Click Allow and relaunch your application to get it running on the device.

