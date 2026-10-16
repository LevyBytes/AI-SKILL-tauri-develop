# Map resource destinations

Open when controlling resource destinations with a path map or choosing glob semantics.

[Topic context](resource-bundle-configuration.md)

For precise command over the destination of each file, supply a map rather than a list:

- Syntax
- Explanation
  tauri.conf.json

```json
{
  "bundle": {
    "resources": {
      "/absolute/path/to/textfile.txt": "resources/textfile.txt",
      "relative/path/to/jsonfile.json": "resources/jsonfile.json",
      "resources/": "",
      "docs/**/*md": "website-docs/"
    }
  }
}
```

tauri.conf.json5

```json5
{
  "bundle": {
    "resources": {
      // `textfile.txt` will be placed to `$RESOURCE/resources/textfile.txt`
      "/absolute/path/to/textfile.txt": "resources/textfile.txt",

      // `jsonfile.json` will be placed to `$RESOURCE/resources/jsonfile.json`
      "relative/path/to/jsonfile.json": "resources/jsonfile.json",

      // Copy the entire directory to `$RESOURCE`, preserving the original structures,
      // the target is "" which means it will be placed directly in the resource directory `$RESOURCE`, for example:
      //   - `resources/file.txt`                -> `$RESOURCE/file.txt`
      //   - `resources/some-folder/config.json` -> `$RESOURCE/some-folder/config.json`
      "resources/": "",

      // When using glob patterns, the behavior is different from the list one,
      // all the matching files will be placed to the target directory without preserving the original file structures
      // for example:
      //   - `docs/index.md`         -> `$RESOURCE/website-docs/index.md`
      //   - `docs/plugins/setup.md` -> `$RESOURCE/website-docs/setup.md`
      "docs/**/*md": "website-docs/"
    }
  }
}
```

For details on how `$RESOURCE` maps to a real location on every platform, consult the reference for [`resource_dir`](https://docs.rs/tauri/latest/tauri/path/struct.PathResolver.html#method.resource_dir)

Source path syntax

Throughout the notes below, the “target resource directory” means either the string following the colon in the map form, or — when the array form is used — the rebuilt version of the source file paths.

- `"dir/file.txt"`: places the single `file.txt` file inside the target resource directory.
- `"dir/"`: pulls every file **and directories** _recursively_ into the target resource directory. Reach for this when you also want the file system layout of your files and folders kept intact.
- `"dir/*"`: grabs all files within the `dir` directory _non-recursively_ (sub-directories will be ignored) and drops them in the target resource directory.
- `"dir/**"`: raises an error, since `**` matches only directories, so it can locate no files.
- `"dir/**/*"`: sweeps up every file beneath `dir` _recursively_ — covering whatever sits directly in `dir/` plus everything within each nested sub-directory — and deposits it all in the target resource directory.
- `"dir/**/**"`: raises an error, since `**` matches only directories, so it can locate no files.
