---
name: tauri-develop
description: "Use when developing Tauri v2 desktop or mobile applications: configuration, Rust commands, frontend events and channels, shared state, resources and sidecars, dependency updates, icons, debugging, plugins, mobile permissions, IPC mocks, Selenium or WebdriverIO tests. Not for Tauri v1."
license: Apache-2.0
metadata:
  last-updated: "2026-10-22"
---

# Context Router

| Topic | Open when | Read |
| --- | --- | --- |
| Configuration Files | Open when running desktop or mobile development, selecting a device or inspector, or editing Tauri configuration, Cargo and package manifests. | [Reference](references/development-configuration-index.md) |
| Rust Commands and Events | Open when invoking Rust commands, passing or returning data, handling errors and async work, accessing Tauri context, or registering frontend and Rust event listeners. | [Rust commands and events](references/rust-commands-events-index.md) |
| Rust Listener Cleanup | Open when ending Rust event subscriptions with unlisten or registering a one-shot listener with once. | [Rust event listener cleanup](references/rust-event-listener-cleanup.md) |
| Calling Frontend | Open when emitting Rust events to the frontend, selecting event targets and payloads, managing listener lifetimes, streaming channels or evaluating JavaScript. | [Reference](references/rust-frontend-communication-index.md) |
| Resources, Sidecar | Open when bundling and resolving resource files or configuring, naming and running sidecar binaries with permitted arguments. | [Reference](references/resources-sidecars-index.md) |
| Managed state | Open when sharing mutable state, choosing mutexes or retrieving exact state types. | [Managed state](references/managed-state-index.md) |
| Dependency updates | Open when updating and synchronizing npm packages and Cargo crates. | [Dependency updates](references/npm-cargo-dependency-updates.md) |
| Application icons | Open when generating app icons or checking platform icon formats. | [Application icons](references/app-icons-index.md) |
| Debugging Tools | Open when debugging Rust or WebView processes, configuring CrabNebula, Neovim or JetBrains tools, or diagnosing Linux graphics failures. | [Reference](references/debugging-tools-index.md) |
| VS Code | Open when configuring VS Code LLDB or the Windows Visual Studio debugger for a Tauri application. | [Reference](references/vscode-tauri-debugging.md) |
| Plugins | Open when initializing Tauri plugins, exposing APIs and commands, handling lifecycle events or defining command permissions and scope schemas. | [Reference](references/plugin-development-index.md) |
| Mobile Plugins | Open when implementing Android or iOS Tauri plugins, mobile commands, arguments, permissions, lifecycle events, Rust bridges or Android memory-page support. | [Reference](references/mobile-plugin-development-index.md) |
| Mocking | Open when mocking Tauri IPC commands, events or windows and cleaning up test mocks. | [Reference](references/frontend-ipc-event-mocks.md) |
| WebDriver Testing | Open when setting up platform WebDrivers, CI, Selenium or WebdriverIO projects and their test-suite lifecycle. | [Reference](references/webdriver-testing-index.md) |

Open the [reference index](references/INDEX.md) or [topic catalog](references/topics.json) when selecting a reference topic.

# Language Translations

[Reference Language Selection](references/languages.md)

# Context For Open Source

Skill reference documentation is interpreted content for use in open source projects. Check against [official documentation sources](https://v2.tauri.app/develop/) when dealing with defects or issues that may come up from interpreting this skill's references.
