# iOS command argument models

Open when decoding nested mobile command arguments in Swift.

[Topic context](mobile-command-argument-models.md)

### iOS

For iOS, you model the arguments with a class conforming to `Decodable`, and nested objects likewise have to adopt the Decodable protocol:

```swift
import Tauri

class OpenAppArgs: Decodable {
  let name: String
  var timeout: Int?
}

class OpenArgs: Decodable {
  let requiredArg: String
  var allowEdit: Bool?
  var quality: UInt8?
  var app: OpenAppArgs?
}

class ExamplePlugin: Plugin {
  @objc public func openCamera(_ invoke: Invoke) throws {
    let args = try invoke.parseArgs(OpenArgs.self)

    invoke.resolve(["path": "/path/to/photo.jpg"])
  }
}
```
