# Linux graphics symptoms and workarounds

[Topic context](debugging-tools-index.md) / [Previous source section](crabnebula-devtools-setup.md) / [Next source section](neovim-tauri-debugging.md)

Context: Tauri v2: Linux graphics symptoms and workarounds. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-linux-graphics-issues"></a> <a id="source-linux-graphics-issues-1"></a> <a id="source-common-symptoms"></a> <a id="source-workarounds"></a> <a id="source-silent-failures-webgl-and-canvas"></a>

## Linux Graphics Issues

### Linux Graphics Issues

Under Linux, rendering in Tauri goes through WebKitGTK. With certain configurations, NVIDIA GPUs being the usual culprit, WebKitGTK and the graphics driver fail to agree, producing everything from an empty window to small visual glitches. The known symptoms and their fixes are gathered on this page. The initial reports are documented at [tauri-apps/tauri#9394](https://github.com/tauri-apps/tauri/issues/9394).

### Common symptoms

- A window appears yet remains empty or all white.
- Flickering occurs in the window, particularly during a resize.
- During a resize the app crashes and prints nothing helpful.
- The console reports `AcceleratedSurfaceDMABuf was unable to construct a complete framebuffer`.
- The console reports `Gdk-Message: Error 71 (Protocol error) dispatching to Wayland display.`

The root cause for the majority of these is the WebKitGTK DMABUF renderer asking for buffer formats that the NVIDIA driver simply will not supply. Upstream conversation can be found on the [WebKitGTK bug tracker](https://bugs.webkit.org/show_bug.cgi?id=261874) and in the [NVIDIA forums](https://forums.developer.nvidia.com/t/geforce-rtx-4070-flickering-issue-when-using-the-dmabuf-renderer-in-webkitgtk/274741).

### Workarounds

Work through these in sequence; the ones near the top preserve hardware acceleration.

1. Confirm that kernel mode setting is active. With NVIDIA drivers predating version 545 you frequently have to pass `nvidia_drm.modeset=1` as a kernel parameter.
2. Set `__NV_DISABLE_EXPLICIT_SYNC=1`. More often than not this clears the Wayland `Error 71` crash and carries no performance penalty.
3. Set `WEBKIT_DISABLE_DMABUF_RENDERER=1`. This resolves both the DMABUF framebuffer error and the `Error 71` crash, though you forfeit the quicker rendering route.
4. Set `WEBKIT_DISABLE_COMPOSITING_MODE=1`. Reserve this as the final option for the silent crash during a resize, since it switches off accelerated compositing altogether.

For testing you can export these from your shell, or assign them inside `main()` ahead of the webview's creation so that your users never need to:

```rust
fn main() {
  // Workaround for WebKitGTK on NVIDIA, see tauri-apps/tauri#9394
  #[cfg(target_os = "linux")]
  std::env::set_var("WEBKIT_DISABLE_DMABUF_RENDERER", "1");

  tauri::Builder::default()
    // ...
}
```

Reserve an unconditional override of this kind for cases where you have confirmed the bug actually hits your app. Otherwise you strip the faster path from everybody, even those whose setups were fine.

### Silent failures: WebGL and canvas

Some issues neither crash nor raise an error at all. WebGL and canvas output may quietly fall onto a sluggish path even as the rest of your app appears healthy. A pair of factors makes spotting this from within your frontend difficult:

- Creating a WebGL2 context still reports success even where a software rasterizer or a slow presentation route sits underneath it. Nothing throws, so there is no error to catch.
- For fingerprinting protection, WebKitGTK hides the WebGL renderer string. On any Linux box `WEBGL_debug_renderer_info` comes back as `Apple GPU`, leaving you no way to learn what genuinely drives the context.

Day to day, the giveaway is laggy input or poor frame rates within WebGL-intensive screens such as terminal emulators, editors, maps, and charts, despite that identical code running smoothly in an ordinary browser. Should your app rely on a WebGL rendering path, supply a non-WebGL fallback for Linux and think about surfacing a setting users can flip, rather than counting on the context to report the truth.

---

