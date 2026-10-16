# Resource example file layout

Open when setting up the bundled translation files used by the resource-reading examples.

[Topic context](resource-reading-access-permissions.md)

### Reading resource files

Here, suppose the goal is to ship some extra i18n json files arranged as follows:

```plaintext
.
├── src-tauri/
│   ├── tauri.conf.json
│   ├── lang/
│   │   ├── de.json
│   │   └── en.json
│   └── ...
└── ...
```

tauri.conf.json

```json
{
  "bundle": {
    "resources": ["lang/*"]
  }
}
```

lang/de.json

```json
{
  "hello": "Guten Tag!",
  "bye": "Auf Wiedersehen!"
}
```
