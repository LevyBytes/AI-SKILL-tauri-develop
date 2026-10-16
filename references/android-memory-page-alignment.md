# Android memory page alignment

Open when configuring native libraries for Android 16KB memory pages.

[Topic context](mobile-rust-interop-alignment.md)

### Android 16KB Memory Pages

Google is in the process of mandating 16KB memory pages for every new Android app submission. Compiling against NDK version 28 or later ought to emit bundles that already satisfy the rule, yet if you are stuck on an older NDK or the produced files end up not 16KB aligned, dropping the following into `.cargo/config.toml` signals it to `rustc`:

```toml
[target.aarch64-linux-android]
rustflags = ["-C", "link-arg=-Wl,-z,max-page-size=16384"]
```
