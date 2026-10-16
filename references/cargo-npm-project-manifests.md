# Cargo and package manifests

[Topic context](development-configuration-index.md) / [Previous source section](platform-configuration-merging.md)

Context: Tauri v2: Cargo and package manifests. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-cargotoml"></a> <a id="source-packagejson"></a>

### `Cargo.toml`

Within Cargo's manifest file you list the Rust crates your app relies on, your app's metadata, and assorted Rust-specific features. Unless you plan to write your app's backend in Rust, you probably won't touch it often, yet it's worth understanding that it's there and what role it plays.

Here is a stripped-down sample `Cargo.toml` file for a Tauri project:

Cargo.toml

```toml
[package]
name = "app"
version = "0.1.0"
description = "A Tauri App"
authors = ["you"]
license = ""
repository = ""
default-run = "app"
edition = "2021"
rust-version = "1.57"

[build-dependencies]
tauri-build = { version = "2.0.0" }

[dependencies]
serde_json = "1.0"
serde = { version = "1.0", features = ["derive"] }
tauri = { version = "2.0.0", features = [ ] }
```

The pieces that deserve the most attention are the `tauri-build` and `tauri` dependencies. As a rule, both ought to sit on the same latest minor versions as the Tauri CLI, although that's not an absolute requirement. Whenever your app misbehaves at runtime, verify that each Tauri version (`tauri` and `tauri-cli`) is current for its respective minor release.

Cargo's version numbers follow [Semantic Versioning](https://semver.org). Invoking `cargo update` from within the `src-tauri` folder fetches the newest Semver-compatible version of every dependency. As an example, declaring `2.0.0` as the version of `tauri-build` leads Cargo to find and grab version `2.0.1`, that being the most recent Semver-compatible release on offer. Tauri only bumps the major version number on a breaking change, so you can count on safely moving up to the latest minor and patch versions without worrying that your code will break.

Should you need to pin a particular crate version, you can require exact versions by putting `=` in front of the dependency's version number:

```plaintext
tauri-build = { version = "=2.0.0" }
```

One more thing worth noticing is the `features=[]` segment of the `tauri` dependency. Both `tauri dev` and `tauri build` figure out on their own which features your project requires, drawing on your Tauri configuration. The [documentation](https://docs.rs/tauri/2.0.0/tauri/#cargo-features) goes deeper on `tauri` feature flags.

Building your application generates a `Cargo.lock` file. Its main job is to guarantee that identical dependencies are used on every machine throughout development (much like `yarn.lock`, `pnpm-lock.yaml` or `package-lock.json` over in Node.js). Committing this file to your source repository is advised so that your builds stay consistent.

For a fuller account of the Cargo manifest file, consult the [official documentation](https://doc.rust-lang.org/cargo/reference/manifest.html).

### `package.json`

This serves as Node.js's package file. When your Tauri app's frontend is built on Node.js-based technologies (think `npm`, `yarn`, or `pnpm`), this file is where you set up the frontend dependencies and scripts.

A minimal `package.json` file for a Tauri project could resemble something along these lines:

package.json

```json
{
  "scripts": {
    "dev": "command to start your app development mode",
    "build": "command to build your app frontend",
    "tauri": "tauri"
  },
  "dependencies": {
    "@tauri-apps/api": "^2.0.0",
    "@tauri-apps/cli": "^2.0.0"
  }
}
```

People often rely on the `"scripts"` section to hold the commands that start and build the frontend powering their Tauri application. In the `package.json` shown above, the `dev` command (run via `yarn dev` or `npm run dev`) boots the frontend framework, while the `build` command (run via `yarn build` or `npm run build`) compiles your frontend's Web assets for Tauri to bundle in production. The handiest way to wire these scripts up is to connect them to the Tauri CLI through the Tauri configuration's [beforeDevCommand](https://v2.tauri.app/reference/config/#beforedevcommand-1) and [beforeBuildCommand](https://v2.tauri.app/reference/config/#beforebuildcommand) hooks:

tauri.conf.json

```json
{
  "build": {
    "beforeDevCommand": "yarn dev",
    "beforeBuildCommand": "yarn build"
  }
}
```

The dependencies object lays out which packages Node.js needs to fetch when you run `yarn`, `pnpm install` or `npm install` — here that means the Tauri CLI and API.

Alongside the `package.json` file you might also find a `yarn.lock`, `pnpm-lock.yaml` or `package-lock.json` file. Their purpose is to make sure that any later install of the dependencies hands you precisely the versions you relied on during development (akin to `Cargo.lock` in Rust).

For more on the `package.json` file format, see the [official documentation](https://docs.npmjs.com/cli/v8/configuring-npm/package-json).

---
