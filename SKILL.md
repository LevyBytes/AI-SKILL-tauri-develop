---
name: tauri-develop
description: Use when developing a Tauri v2 desktop or mobile app: project configuration (tauri.conf.json), calling Rust commands from the frontend and calling the frontend from Rust (commands, events, channels), state management, bundling resources and sidecar binaries, app icons, updating dependencies, debugging (VS Code, RustRover, Neovim, Linux graphics, CrabNebula DevTools), writing and developing plugins (including mobile), and testing (mocking, WebDriver with Selenium/WebdriverIO). Covers the Develop section of the Tauri v2 docs. Not for Tauri v1.
covers:
  - configuration-files
  - calling-rust
  - calling-frontend
  - resources-sidecar
  - vscode
  - plugins
  - develop-mobile
  - mocking
  - ci-example
---

# Tauri v2 - Develop Reference

Faithful reference for Tauri v2 - Develop, split into one focused file per subject across 12 reference files.
Identifiers (API/type/function names, flags, paths, enums, numbers) are preserved verbatim.

## When to use this

Use this skill when the task involves developing a Tauri v2 app - tauri.conf.json configuration, calling Rust/frontend, state management, resources/sidecar, icons, debugging, plugin development, and testing.

## Workflow

1. Open `references/INDEX.md` (or `references/topics.json`) and pick the one file that matches.
2. Read only that file; open another only if the task spans subjects. Grep across files when needed: `rg -n "PATTERN" references/*.md`.
3. Treat every identifier as an exact reference fact — never invent or paraphrase.

## Gotchas

Recurring failure modes and what to do instead live in the sibling [GOTCHA.md](GOTCHA.md).

## References

All depth lives in `references/` — start at `references/INDEX.md`, metadata in `references/topics.json`. One subject per file.

## Official documentation

The official documentation is at https://v2.tauri.app/develop/ — consult it to verify anything uncertain, conflicting, or version-specific.

## Verification

```powershell
python .agents/skills/skill-drafting/scripts/validate_skill_package.py .agents/skills/tauri-develop
```
