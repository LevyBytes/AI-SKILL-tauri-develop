# Neovim debug server and keys

Open when starting the frontend server or binding Neovim debugging commands.

[Topic context](neovim-tauri-debugging.md)

### Starting the dev server

Because the Tauri CLI isn't the thing starting our app here, the development server won't come up by itself. The [overseer](https://github.com/stevearc/overseer.nvim/tree/master) plugin lets you drive the development server's state straight from Neovim.

For managing tasks that run in the background, your best bet is a [VS Code style task](https://github.com/stevearc/overseer.nvim/blob/master/doc/guides.md#vs-code-tasks) configuration. Set this up by adding a `.vscode/tasks.json` file within the project directory.

A sample task configuration for a project built around `trunk` appears below.

.vscode/tasks.json

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "type": "process",
      "label": "dev server",
      "command": "trunk",
      "args": ["serve"],
      "isBackground": true,
      "presentation": {
        "revealProblems": "onProblem"
      },
      "problemMatcher": {
        "pattern": {
          "regexp": "^error:.*",
          "file": 1,
          "line": 2
        },
        "background": {
          "activeOnStart": false,
          "beginsPattern": ".*Rebuilding.*",
          "endsPattern": ".*server listening at:.*"
        }
      }
    }
  ]
}
```

### Example key bindings

Sample key bindings for launching and steering debugging sessions follow.

init.lua

```lua
vim.keymap.set('n', '<F5>', function() dap.continue() end)
vim.keymap.set('n', '<F6>', function() dap.disconnect({ terminateDebuggee = true }) end)
vim.keymap.set('n', '<F10>', function() dap.step_over() end)
vim.keymap.set('n', '<F11>', function() dap.step_into() end)
vim.keymap.set('n', '<F12>', function() dap.step_out() end)
vim.keymap.set('n', '<Leader>b', function() dap.toggle_breakpoint() end)
vim.keymap.set('n', '<Leader>o', function() overseer.toggle() end)
vim.keymap.set('n', '<Leader>R', function() overseer.run_template() end)
```

---

Load the [debug adapter configuration](neovim-debug-adapter-setup.md) first; these key bindings use its local dap and dapui variables.
