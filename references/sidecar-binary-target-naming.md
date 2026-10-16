# Sidecar binary configuration and target naming

[Topic context](resources-sidecars-index.md) / [Previous source section](resource-reading-access-permissions.md) / [Next source section](sidecar-execution-argument-permissions.md)

Context: Tauri v2: Sidecar binary configuration and target naming. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-embedding-external-binaries"></a> <a id="source-embedding-external-binaries-1"></a>

## Embedding External Binaries

### Embedding External Binaries

There are times when bundling external binaries makes sense — to extend what your application can do, or to spare users from having to install further dependencies themselves (Node.js or Python, say). Tauri calls such a binary a `sidecar`.

A binary here is just an executable, written in whatever programming language you like. Typical examples include Python CLI tools or API servers packaged with `pyinstaller`.

Register the binaries you want shipped by adding the `externalBin` property onto the `bundle` object inside your `tauri.conf.json`. What `externalBin` accepts is a list of strings pointing at binaries via either absolute or relative paths.

The Tauri configuration fragment below shows what a sidecar setup looks like:

src-tauri/tauri.conf.json

```json
{
  "bundle": {
    "externalBin": [
      "/absolute/path/to/sidecar",
      "../relative/path/to/binary",
      "binaries/my-sidecar"
    ]
  }
}
```

For the external binary to run across every architecture you support, a same-named binary carrying a `-$TARGET_TRIPLE` suffix has to be present at the given path. As an example, `"externalBin": ["binaries/my-sidecar"]` calls for a `src-tauri/binaries/my-sidecar-x86_64-unknown-linux-gnu` executable on Linux, or `src-tauri/binaries/my-sidecar-aarch64-apple-darwin` on Mac OS running Apple Silicon.

Figure out your **current** platform's `-$TARGET_TRIPLE` suffix by issuing this command:

Terminal window

```sh
rustc --print host-tuple
```

That prints your host's target triple straight to the console (for example `x86_64-unknown-linux-gnu` or `aarch64-apple-darwin`).

The Node.js script below tacks the target triple onto a binary:

```javascript
import { execSync } from 'child_process';
import fs from 'fs';

const extension = process.platform === 'win32' ? '.exe' : '';

const targetTriple = execSync('rustc --print host-tuple').toString().trim();
if (!targetTriple) {
  console.error('Failed to determine platform target triple');
}
fs.renameSync(
  `src-tauri/binaries/sidecar${extension}`,
  `src-tauri/binaries/sidecar-${targetTriple}${extension}`
);
```

Bear in mind the script breaks down once you compile for an architecture other than the one it executes on, so treat it merely as a jumping-off point for your own build scripts.

