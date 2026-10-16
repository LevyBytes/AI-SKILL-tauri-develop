# Update and synchronize npm and Cargo dependencies

<a id="updating-dependencies"></a> <a id="updating-dependencies-1"></a> <a id="update-npm-packages"></a> <a id="update-cargo-packages"></a> <a id="sync-npm-packages-and-cargo-crates-versions"></a>

[Topic context](INDEX.md) / [Previous source section](managed-state-access-types.md) / [Next source section](app-icon-generation.md)

Context: Tauri v2: Update and synchronize npm and Cargo dependencies. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-updating-dependencies"></a> <a id="source-updating-dependencies-1"></a> <a id="source-update-npm-packages"></a> <a id="source-update-cargo-packages"></a> <a id="source-sync-npm-packages-and-cargo-crates-versions"></a>

## Updating Dependencies

### Updating Dependencies

### Update npm Packages

For projects that depend on the `tauri` package:

- npm
- yarn
- pnpm

```sh
npm install @tauri-apps/cli@latest @tauri-apps/api@latest
```

```sh
yarn up @tauri-apps/cli @tauri-apps/api
```

```sh
pnpm update @tauri-apps/cli @tauri-apps/api --latest
```

From a terminal, you can likewise discover which Tauri release is currently newest by running:

- npm
- yarn
- pnpm

```sh
npm outdated @tauri-apps/cli
```

```sh
yarn outdated @tauri-apps/cli
```

```sh
pnpm outdated @tauri-apps/cli
```

### Update Cargo Packages

To find packages that have fallen behind, lean on [`cargo outdated`](https://github.com/kbknapp/cargo-outdated), or consult the crates.io listings directly: [tauri](https://crates.io/crates/tauri/versions) / [tauri-build](https://crates.io/crates/tauri-build/versions).

Open `src-tauri/Cargo.toml` and set `tauri` plus `tauri-build` to

```toml
[build-dependencies]
tauri-build = "%version%"

[dependencies]
tauri = { version = "%version%" }
```

in which `%version%` stands for the matching version number sourced above.

After that, run:

Terminal window

```shell
cd src-tauri
cargo update
```

As another route, the `cargo upgrade` command that ships with [cargo-edit](https://github.com/killercup/cargo-edit) performs every one of these steps for you.

### Sync npm Packages and Cargo Crates versions

Because those JavaScript APIs sit on top of Rust code running in the backend, shipping a new capability means raising both halves in step so they stay compatible. Take care that the minor version of the npm package `@tauri-apps/api` lines up with the cargo crate `tauri`

Plugins behave a little differently: changes of this kind can arrive in patch releases, so the npm package and cargo crate versions advance in lockstep and their exact values must remain identical — to give one case, the npm package `@tauri-apps/plugin-fs` and the cargo crate `tauri-plugin-fs` should both sit at the same version (say `2.2.1`)

---

