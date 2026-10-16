# Bundle a resource list

Open when bundling files, directories or globs while preserving source path structure.

[Topic context](resource-bundle-configuration.md)

# Resources, Sidecar

> Tauri Develop reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Embedding Additional Files; Embedding External Binaries.

## Embedding Additional Files

### Embedding Additional Files

Sometimes an app bundle has to ship extra files that don't belong to the frontend (your `frontendDist`) or that are simply too large to embed straight into the binary. Tauri refers to such files as `resources`.

### Configuration

Pick the files you want shipped and register them through the `resources` property on the `bundle` object within your `tauri.conf.json` file.

Supplying them as a list of paths:

- Syntax
- Explanation
  tauri.conf.json

```json
{
  "bundle": {
    "resources": [
      "./path/to/some-file.txt",
      "/absolute/path/to/textfile.txt",
      "../relative/path/to/jsonfile.json",
      "some-folder/",
      "resources/**/*.md"
    ]
  }
}
```

tauri.conf.json5

```json5
{
  "bundle": {
    "resources": [
      // Will be placed to `$RESOURCE/path/to/some-file.txt`
      "./path/to/some-file.txt",

      // The root in an absolute path will be replaced by `_root_`,
      // so `textfile.txt` will be placed to `$RESOURCE/_root_/absolute/path/to/textfile.txt`
      "/absolute/path/to/textfile.txt",

      // `..` in a relative path will be replaced by `_up_`,
      // so `jsonfile.json` will be placed to `$RESOURCE/_up_/relative/path/to/jsonfile.json`,
      "../relative/path/to/jsonfile.json",

      // If the path is a directory, the entire directory will be copied to the `$RESOURCE` directory,
      // preserving the original structures, for example:
      //   - `some-folder/file.txt`                   -> `$RESOURCE/some-folder/file.txt`
      //   - `some-folder/another-folder/config.json` -> `$RESOURCE/some-folder/another-folder/config.json`
      // This is the same as `some-folder/**/*`
      "some-folder/",

      // You can also include multiple files at once through glob patterns.
      // All the `.md` files inside `resources` will be placed to `$RESOURCE/resources/`,
      // preserving their original directory structures, for example:
      //   - `resources/index.md`      -> `$RESOURCE/resources/index.md`
      //   - `resources/docs/setup.md` -> `$RESOURCE/resources/docs/setup.md`
      "resources/**/*.md"
    ]
  }
}
```

Whatever you bundle lands under `$RESOURCE/` and keeps its original folder layout intact — so, for instance: `./path/to/some-file.txt` -> `$RESOURCE/path/to/some-file.txt`
