# Build configuration variants

Open when selecting an alternate app name or identifier through a CLI configuration override.

[Topic context](platform-configuration-merging.md)

### Extending the Configuration

When you invoke any of the `dev`, `android dev`, `ios dev`, `build`, `android build`, `ios build` or `bundle` commands, the Tauri CLI lets you augment the Tauri configuration. You pass this extension through the `--config` argument, supplying either a raw JSON string or a path pointing to a JSON file. To combine the value you give with the configuration object that was originally resolved, Tauri applies the [JSON Merge Patch (RFC 7396)](https://datatracker.ietf.org/doc/html/rfc7396) specification.

You can lean on this approach to spin up several variants of your application or to gain extra leeway in how you configure your application bundles.

To take one case, shipping a fully separate _beta_ application is possible by using this feature to set a distinct application name and identifier:

src-tauri/tauri.beta.conf.json

```json
{
  "productName": "My App Beta",
  "identifier": "com.myorg.myappbeta"
}
```

Then, to ship that standalone _beta_ app, hand over this configuration file at build time:

- npm
- yarn
- pnpm
- deno
- bun
- cargo

```sh
npm run tauri build -- --config src-tauri/tauri.beta.conf.json
```

```sh
yarn tauri build --config src-tauri/tauri.beta.conf.json
```

```sh
pnpm tauri build --config src-tauri/tauri.beta.conf.json
```

```sh
deno task tauri build --config src-tauri/tauri.beta.conf.json
```

```sh
bun tauri build --config src-tauri/tauri.beta.conf.json
```

```sh
cargo tauri build --config src-tauri/tauri.beta.conf.json
```
