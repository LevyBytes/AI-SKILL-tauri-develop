# Set up the Neovim debugger

Open when configuring nvim-dap, its UI and breakpoint signs for Tauri.

[Topic context](neovim-tauri-debugging.md)

## Debug in Neovim

### Debug in Neovim

Quite a few plugins exist for debugging Rust within Neovim. What follows walks you through configuring `nvim-dap` along with a couple of companion plugins so you can debug a Tauri application.

### Prerequisites

The `nvim-dap` extension depends on the `codelldb` binary. Grab the build that matches your platform from [https://github.com/vadimcn/codelldb/releases](https://github.com/vadimcn/codelldb/releases) and extract the archive. The `nvim-dap` configuration will reference it shortly.

### Configuring nvim-dap

Add the [`nvim-dap`](https://github.com/mfussenegger/nvim-dap) and [`nvim-dap-ui`](https://github.com/rcarriga/nvim-dap-ui) plugins. Either follow what their github pages describe or just lean on whichever plugin manager you prefer. Keep in mind that `nvim-dap-ui` depends on the `nvim-nio` plugin.

After that, wire the plugin into your Neovim configuration:

init.lua

```lua
local dap = require("dap")

dap.adapters.codelldb = {
  type = 'server',
  port = "${port}",
  executable = {
    -- Change this to your path!
    command = '/opt/codelldb/adapter/codelldb',
    args = {"--port", "${port}"},
  }
}

dap.configurations.rust= {
  {
    name = "Launch file",
    type = "codelldb",
    request = "launch",
    program = function()
      return vim.fn.input('Path to executable: ', vim.fn.getcwd() .. '/target/debug/', 'file')
    end,
    cwd = '${workspaceFolder}',
    stopOnEntry = false
  },
}
```

With this arrangement, every time you start the debugger it prompts you to indicate which Tauri App binary you intend to debug.

If you like, configure the `nvim-dap-ui` plugin so the debugger view opens and closes on its own whenever a debugging session begins or ends:

init.lua

```lua
local dapui = require("dapui")
dapui.setup()

dap.listeners.before.attach.dapui_config = function()
  dapui.open()
end
dap.listeners.before.launch.dapui_config = function()
  dapui.open()
end
dap.listeners.before.event_terminated.dapui_config = function()
  dapui.close()
end
dap.listeners.before.event_exited.dapui_config = function()
  dapui.close()
end
```

Finally, the editor's standard rendering of breakpoints can be customized:

init.lua

```lua
vim.fn.sign_define('DapBreakpoint',{ text ='🟥', texthl ='', linehl ='', numhl =''})
vim.fn.sign_define('DapStopped',{ text ='▶️', texthl ='', linehl ='', numhl =''})
```
