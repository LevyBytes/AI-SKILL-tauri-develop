# Web surfaces in Tauri applications

Identify the window or WebView, operating system, engine version, origin and capabilities for the failing interaction. Windows uses WebView2, Android uses its WebView provider, macOS and iOS use WKWebView, and Linux uses WebKitGTK. Reproduce the issue in the affected application. [WebView versions](https://v2.tauri.app/reference/webview-versions/) · [WebView diagnostics](webview-console-devtools.md)

- **Main windows, dialogs, settings and secondary windows.** Start with semantic controls, meaningful focus order, dialog dismissal and restoration, accessible status messages, long translations and text scaling. Resize a window with a form or menu open. Check keyboard-only use and reduced motion. [Accessibility](accessible-interfaces/index.md) · [Components](interface-components/index.md) · [Interaction patterns](interaction-patterns/index.md)
- **Mobile WebViews.** Test the virtual keyboard, IME composition, rotation, touch targets and scroll containment on the target OS. Keep the primary action reachable with the keyboard open. Check permission denial and resume before showing a success state. [Forms](form-controls/index.md) · [Adaptive experiences](adaptive-experiences/index.md) · [Mobile permission handling](mobile-permission-requests.md)
- **Web content and native commands.** Render plain text as text and handle intentionally rich content separately. Constrain document resources with CSP and grant native commands only to the windows or WebViews that need them. Validate inputs in Rust. Review remote origins and frames separately. [Browser security](browser-security/index.md) · [Tauri capabilities](https://v2.tauri.app/security/capabilities/) · [Tauri CSP](https://v2.tauri.app/security/csp/) · [Command scopes](plugin-command-global-scopes.md)
- **State, IPC and rendering.** Separate transient DOM state from application state. Keep a pending edit through rerender or window recreation where the product requires it. Target the correct window for progress events and release listeners when views close. Measure renderer work separately from IPC transfer and Rust execution. [Client data](client-data/index.md) · [Managed state](managed-state-index.md) · [Event targets](rust-webview-event-targets.md) · [Rendering performance](rendering-performance/index.md)
- **Desktop and mobile OS integration.** Treat offline drafts, file opening, sharing, notifications and draggable title bars as user outcomes. Pick a native window API, capability-scoped plugin or supported WebView API per target. PWA install prompts, service-worker updates and web app manifests describe a different distribution path. For a custom title bar test interactive buttons inside drag regions. [Integration topics](installed-web-apps/index.md) · [Resources](resources-sidecars-index.md) · [Mobile plugins](mobile-plugin-development-index.md)
- **Optional browser features.** Detect APIs in the actual document. Keep a fallback for unsupported CSS, local models, passkeys, geolocation, capture, clipboard or WebAssembly. A browser example does not grant a native permission. For WebMCP, deliberately select what document actions may be exposed and keep native capabilities scoped. [Feature support](feature-support/index.md) · [Browser privacy](privacy-controls/index.md)
- **Verification.** Exercise layout and accessibility in an isolated renderer. Use IPC mocks for command names, arguments, errors and listener cleanup. Run a real app for capability denial, multiwindow routing, native dialogs and platform behavior. Tauri documents an embedded WebdriverIO service option on macOS. Driving the upstream native tauri-driver directly supports Windows and Linux on desktop. [IPC mocks](frontend-ipc-event-mocks.md) · [App tests](webdriver-testing-index.md) · [Current WebDriver options](https://v2.tauri.app/develop/tests/webdriver/)

## Local topic references

- Open when checking accessible names, error announcements or system text scaling. [Accessible interfaces](accessible-interfaces/index.md).
- Open when adapting navigation and interaction to devices and user preferences. [Adaptive experiences](adaptive-experiences/index.md).
- Open when checking device support for prompting, translation, detection or summarization. [Built-in model APIs](browser-models/index.md).
- Open when implementing origin checks, content safety, cryptography or passkeys. [Browser security and identity](browser-security/index.md).
- Open when working with client data, timestamps, calendars or asynchronous state. [JavaScript and time](client-data/index.md).
- Open when choosing semantic elements, creating custom elements or managing a shadow tree. [HTML and web components](document-structure/index.md).
- Open when checking platform feature identifiers, trial records or fallback behavior. [Feature support and fallbacks](feature-support/index.md).
- Open when implementing validation, autofill, text entry, pickers or range controls. [Form entry and selection](form-controls/index.md).
- Open when building offline behavior, installation, updates or OS integration. [Installed web applications](installed-web-apps/index.md).
- Open when handling gestures, navigation, focus, motion or scroll behavior. [Interaction patterns](interaction-patterns/index.md).
- Open when building navigation, dialogs, menus, tables or application controls. [Interface components](interface-components/index.md).
- Open when positioning overlays or adding scroll cues, icons and overflow indicators. [Interface details](interface-details/index.md).
- Open when choosing layout, intrinsic sizing, conditional styles or responsive components. [CSS layout and the cascade](layout-and-cascade/index.md).
- Open when handling web identity, embedded state and privacy boundaries. [Browser privacy](privacy-controls/index.md).
- Open when measuring loading, rendering, interaction latency or resource priorities. [Loading and responsiveness](rendering-performance/index.md).
- Open when selecting typography, color, artwork and visual treatments. [Visual presentation](visual-presentation/index.md).
- Open when exposing a form or registering an authorized tool from a web document. [WebMCP tools](web-tools/index.md).
- Open when compiling C or C++ for a browser runtime. [WebAssembly builds](webassembly-builds/index.md).

Use Node.js 22.18 or later to search the local web guides from this skill directory. Run `node scripts/lookup.ts search "IME composing text"` and then `node scripts/lookup.ts retrieve ime-safe-enter-submit`. Use this local command where captured prose refers to the upstream search package. Select `--edition published` for expanded examples when available. Web lookup covers the local web guides. The product reference index covers the host APIs.
