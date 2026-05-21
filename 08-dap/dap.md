# DAP — Neovim 调试器配置

> **TL;DR**: nvim-dap 将 VS Code 的调试体验带入 Neovim。nvim-dap-ui 提供 IDE 级调试界面。通过 mason-nvim-dap 可自动安装调试适配器。

---

## 1. 核心插件生态

| 插件 | 职责 |
|------|------|
| **nvim-dap** | DAP 客户端核心（调试器协议实现） |
| **nvim-dap-ui** | 侧边栏调试界面（变量、堆栈、断点、REPL） |
| **nvim-dap-virtual-text** | 代码行内变量值显示 |
| **mason-nvim-dap** | 通过 Mason 安装调试适配器 |

---

## 2. 基础配置

```lua
-- lua/plugins/dap.lua
return {
  -- 调试器核心
  {
    "mfussenegger/nvim-dap",
    dependencies = {
      "nvim-neotest/nvim-nio", -- 异步 I/O 库
      -- Mason DAP（自动安装适配器）
      {
        "jay-babu/mason-nvim-dap.nvim",
        dependencies = { "mason-org/mason.nvim" },
        opts = {
          ensure_installed = { "python", "codelldb" },
        },
      },
    },
    keys = {
      { "<F5>", function() require("dap").continue() end, desc = "继续/启动调试" },
      { "<F10>", function() require("dap").step_over() end, desc = "单步跳过" },
      { "<F11>", function() require("dap").step_into() end, desc = "单步进入" },
      { "<F12>", function() require("dap").step_out() end, desc = "单步跳出" },
      { "<leader>db", function() require("dap").toggle_breakpoint() end, desc = "切换断点" },
      { "<leader>dB", function()
          require("dap").set_breakpoint(vim.fn.input("断点条件: "))
        end, desc = "条件断点" },
      { "<leader>dr", function() require("dap").repl.open() end, desc = "打开 REPL" },
      { "<leader>dl", function() require("dap").run_last() end, desc = "运行上次" },
    },
  },

  -- 调试 UI 界面
  {
    "rcarriga/nvim-dap-ui",
    dependencies = { "mfussenegger/nvim-dap", "nvim-neotest/nvim-nio" },
    keys = {
      { "<leader>du", function() require("dapui").toggle() end, desc = "切换调试 UI" },
    },
    config = function()
      local dap = require("dap")
      local dapui = require("dapui")
      dapui.setup()

      -- 调试开始时自动打开 UI
      dap.listeners.before.attach.dapui_config = function() dapui.open() end
      dap.listeners.before.launch.dapui_config = function() dapui.open() end
      -- 调试结束时自动关闭 UI
      dap.listeners.before.event_terminated.dapui_config = function() dapui.close() end
      dap.listeners.before.event_exited.dapui_config = function() dapui.close() end
    end,
  },

  -- 内联变量值显示
  {
    "theHamsta/nvim-dap-virtual-text",
    opts = {
      enabled = true,
      highlight_changed_variables = true,
      commented = false,
    },
  },
}
```

> 来源：[nvim-dap README](https://github.com/mfussenegger/nvim-dap) | [nvim-dap-ui README](https://github.com/rcarriga/nvim-dap-ui)

---

## 3. 各语言调试适配器配置

### Python（debugpy）

```lua
-- 使用 nvim-dap-python 扩展（推荐）
{
  "mfussenegger/nvim-dap-python",
  config = function()
    require("dap-python").setup("python3")
  end,
}
```

或手动配置：

```lua
local dap = require("dap")
dap.adapters.python = {
  type = "executable",
  command = "python3",
  args = { "-m", "debugpy.adapter" },
}
dap.configurations.python = {
  {
    type = "python",
    request = "launch",
    name = "Launch file",
    program = "${file}",
  },
}
```

### JavaScript / TypeScript（vscode-js-debug）

```lua
dap.adapters["pwa-node"] = {
  type = "server",
  host = "localhost",
  port = "${port}",
  executable = {
    command = "node",
    args = {
      vim.fn.stdpath("data") .. "/mason/packages/js-debug-adapter/js-debug/src/dapDebugServer.js",
      "${port}",
    },
  },
}
dap.configurations.javascript = {
  {
    name = "Launch file",
    type = "pwa-node",
    request = "launch",
    program = "${file}",
    cwd = "${workspaceFolder}",
  },
}
```

### Rust（CodeLLDB）

```lua
dap.adapters.codelldb = {
  type = "server",
  port = "${port}",
  executable = {
    command = vim.fn.stdpath("data") .. "/mason/packages/codelldb/extension/adapter/codelldb",
    args = { "--port", "${port}" },
  },
}
dap.configurations.rust = {
  {
    name = "Launch",
    type = "codelldb",
    request = "launch",
    program = function()
      return vim.fn.input("可执行文件路径: ", vim.fn.getcwd() .. "/target/debug/", "file")
    end,
    cwd = "${workspaceFolder}",
    stopOnEntry = false,
  },
}
```

### Go（Delve）

```lua
dap.adapters.delve = {
  type = "server",
  port = "${port}",
  executable = {
    command = "dlv",
    args = { "dap", "-l", "127.0.0.1:${port}" },
  },
}
dap.configurations.go = {
  {
    type = "delve",
    name = "Debug",
    request = "launch",
    program = "${file}",
  },
}
```

---

## 4. which-key 调试分组

```lua
local wk = require("which-key")
wk.add({
  { "<leader>d", group = "Debug / 调试" },
  { "<leader>db", desc = "切换断点" },
  { "<leader>dB", desc = "条件断点" },
  { "<leader>dc", desc = "继续" },
  { "<leader>dn", desc = "单步跳过" },
  { "<leader>di", desc = "单步进入" },
  { "<leader>do", desc = "单步跳出" },
  { "<leader>dr", desc = "打开 REPL" },
  { "<leader>du", desc = "切换调试 UI" },
})
```

---

## 5. 关键参考

| 资源 | URL |
|------|-----|
| nvim-dap | https://github.com/mfussenegger/nvim-dap |
| nvim-dap 官方文档 | https://github.com/mfussenegger/nvim-dap/blob/master/doc/dap.txt |
| nvim-dap-ui | https://github.com/rcarriga/nvim-dap-ui |
| nvim-dap-virtual-text | https://github.com/theHamsta/nvim-dap-virtual-text |
| mason-nvim-dap | https://github.com/jay-babu/mason-nvim-dap.nvim |
| Debug Adapter 安装 Wiki | https://codeberg.org/mfussenegger/nvim-dap/wiki/Debug-Adapter-installation |
