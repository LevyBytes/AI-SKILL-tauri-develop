# Tauri v2 Develop Reference Skill

A reference skill for **[Agent Kaizen](https://github.com/DirectorGunner/agent-kaizen)** covering the Develop section of the Tauri v2 documentation, for building Tauri v2 desktop and mobile apps. The prose is original (recontextualized); identifiers — API/type/function names, flags, paths, enums, numbers, and code — are preserved exactly.

## What this skill covers

This skill is a faithful reference for the Develop section of the Tauri v2 documentation, for building Tauri v2 desktop and mobile apps. It covers project configuration (`tauri.conf.json`), calling Rust commands from the frontend and the frontend from Rust (commands, events, channels), state management, bundling resources and sidecar binaries, app icons, updating dependencies, and debugging (VS Code, RustRover, Neovim, Linux graphics, CrabNebula DevTools). It also covers writing and developing plugins — including mobile plugins — and testing via mocking and WebDriver (Selenium / WebdriverIO). It is scoped to Tauri v2 and is not for Tauri v1.

## What's inside

- `SKILL.md` — the skill entry point: frontmatter trigger, when-to-use, workflow, and verification.
- `references/` — right-sized topic files (one subject per file: configuration, calling Rust, calling the frontend, resources/sidecar, debugging, VS Code, plugins, mobile development, tests, WebDriver), plus `INDEX.md` and `topics.json` for routing.
- `GOTCHA.md` — recurring failure modes and what to do instead.

## Use it

This skill is one git repo inside the Agent Kaizen **skills store**. The store nests two folders on purpose: the outer **`SKILLS\`** is a VS Code project for building and maintaining skills (its own workspace + tooling), and the inner lowercase **`skills\`** holds every skill as its own repo. That split lets a project pull skills two ways — the **whole `skills\` folder at once** (loads everything — **not recommended**) or **one skill at a time** (recommended: load only what a task needs and stay under Claude Code's skill-listing budget).

Paths below use **`%DEVROOT%`** — the `DEVROOT` environment variable pointing at the folder that contains `SKILLS\`. Set it once by running **`SetDevRoot.cmd`** in the SKILLS repo root (no admin); a new shell then resolves `%DEVROOT%` automatically.

Link **just this skill** (recommended) — a Windows directory junction, no admin:

```cmd
mklink /J .agents\skills\tauri-develop  "%DEVROOT%\SKILLS\skills\tauri-develop"
mklink /J .claude\skills\tauri-develop  "%DEVROOT%\SKILLS\skills\tauri-develop"
```

Or link the **whole store** at once (loads every skill — not recommended outside a skills-dev project):

```cmd
mklink /J .agents\skills  "%DEVROOT%\SKILLS\skills"
mklink /J .claude\skills  "%DEVROOT%\SKILLS\skills"
```

Remove a link (the store copy is untouched):

```cmd
rmdir .agents\skills\tauri-develop
rmdir .claude\skills\tauri-develop
```

The agent (OpenAI Codex, Claude Code) then auto-loads this skill whenever a task matches its triggers. Built and validated with **[skill-drafting](https://github.com/DirectorGunner/AI-skill-drafting)** to the Agent Kaizen gold standard.

## Status

Recontextualized into original prose (identifiers preserved); packaged in the skills store and ready to publish as the public `AI-SKILL-tauri-develop` repository. Publication is performed by the maintainer.

## Reducing idle context cost (skill invocation policy)

Every installed skill costs a little context on every session: the agent sees each skill's name and description before you ever use it. If you rarely use this skill in a given project, you can keep it installed and still explicitly invocable while hiding it from the model's automatic listing.

Doing so does **not** modify this skill's source repo — the policy lives in your local agent settings. The Agent Kaizen **[skill-drafting](https://github.com/DirectorGunner/AI-skill-drafting)** repo ships a manager, `scripts/skill_policy.py`, that sets this for all your skills at once:

```bash
python skill_policy.py audit     # list every skill + its current policy + idle cost
python skill_policy.py plan      # write a decision file with recommendations (nothing applied)
# edit that decision file: set selected_policy + approved:true for the skills you choose
python skill_policy.py preview   # show the exact change
python skill_policy.py apply     # apply ONLY what you approved (backup + rollback recorded)
python skill_policy.py restore   # roll back
```

- **Claude Code (works today):** writes `skillOverrides: { "<skill>": "user-invocable-only" }` to `.claude/settings.local.json` — zero idle listing cost, still available from the `/skills` menu. Start a new session for it to take effect; invoke it any time via `/skills`.
- **Codex (currently unreliable):** explicit-only Codex skills are affected by an open bug ([openai/codex#23454](https://github.com/openai/codex/issues/23454)) where `$skill` invocation of an explicit-only local skill can fail. Until it's fixed, leave Codex skills implicit, or fully disable rarely-used ones with `[[skills.config]] enabled = false` in `config.toml`. The manager audits Codex but does not change Codex policy.
