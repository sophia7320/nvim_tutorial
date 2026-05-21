# nvim-dap 各语言调试适配器 — 逐行深度配置

> **TL;DR**: 不同语言的 DAP 适配器配置完全不同。逐句提供 Python（debugpy）、JS/TS（vscode-js-debug）、Rust（CodeLLDB）、Go（Delve）的完整配置。

> **前置阅读**：nvim-dap 核心概念（adapter/configuration/UI）和基础快捷键，见当前目录旧版内容 / [08-dap/dap.md](dap.md)。

---

## 1. Python — debugpy

```lua
-- lua/config/dap/python.lua

local dap = require("dap")  -- (1) 加载 nvim-dap

-- ─── 适配器：告诉 nvim-dap 如何启动 debugpy ──────────
dap.adapters.python = {
  type = "executable",              -- (2) 启动子进程，通过 stdio 通信
  command = "python3",              -- (3) Python 解释器路径
  args = { "-m", "debugpy.adapter" }, -- (4) 以模块方式运行 debugpy 适配器
  -- 等价于 shell: python3 -m debugpy.adapter
}

-- ─── 配置：告诉 debugpy 如何运行你的代码 ──────────────
dap.configurations.python = {
  {
    type = "python",                -- (5) 必须匹配适配器键名
    request = "launch",             -- (6) 启动新进程
    name = "Launch file",           -- (7) 选择菜单显示名
    program = "${file}",            -- (8) ${file} = 当前文件路径
    console = "integratedTerminal", -- (9) 输出到 Neovim 终端
    justMyCode = false,             -- (10) false: 可进入第三方库代码

    -- (11) pythonPath — 动态选择 Python 解释器
    pythonPath = function()
      local cwd = vim.fn.getcwd()
      if vim.fn.executable(cwd .. "/venv/bin/python") == 1 then
        return cwd .. "/venv/bin/python"    -- .venv/
      elseif vim.fn.executable(cwd .. "/.venv/bin/python") == 1 then
        return cwd .. "/.venv/bin/python"   -- ./venv/
      end
      return "python3"                      -- 系统默认
    end,
  },
}
```

---

## 2. JavaScript / TypeScript — vscode-js-debug

```lua
-- lua/config/dap/javascript.lua

local dap = require("dap")

-- (12) 适配器：server 模式 — nvim-dap 先启动 node 服务器再 TCP 连接
dap.adapters["pwa-node"] = {
  type = "server",
  host = "localhost",
  port = "${port}",            -- (13) nvim-dap 自动分配端口

  executable = {
    command = "node",
    args = {
      -- (14) Mason 安装的 js-debug DAP 服务器入口
      vim.fn.stdpath("data")
        .. "/mason/packages/js-debug-adapter/js-debug/src/dapDebugServer.js",
      "${port}",
    },
  },
}

dap.configurations.javascript = {
  {
    type = "pwa-node",
    request = "launch",
    name = "Launch file",
    program = "${file}",
    cwd = "${workspaceFolder}",
  },
  -- (15) attach 模式：连接已运行进程
  {
    type = "pwa-node",
    request = "attach",
    name = "Attach to process",
    port = 9229,               -- (16) Node.js 默认调试端口
    restart = true,            -- (17) 断开后自动重连
    sourceMaps = true,         -- (18) TS → JS source map
    protocol = "inspector",    -- (19) V8 Inspector 协议
  },
}
-- (20) TS/TSX 复用 JS 配置
dap.configurations.typescript = dap.configurations.javascript
```

---

## 3. Rust — CodeLLDB

```lua
-- lua/config/dap/rust.lua

local dap = require("dap")

dap.adapters.codelldb = {
  type = "server",
  port = "${port}",
  executable = {
    -- (21) Mason 安装的 codelldb 适配器
    command = vim.fn.stdpath("data")
      .. "/mason/packages/codelldb/extension/adapter/codelldb",
    args = { "--port", "${port}" },
  },
}

dap.configurations.rust = {
  {
    type = "codelldb",
    request = "launch",
    name = "Launch",
    -- (22) 手动指定编译产物路径 (target/debug/<name>)
    program = function()
      return vim.fn.input(
        "Path to executable: ",
        vim.fn.getcwd() .. "/target/debug/",
        "file"
      )
    end,
    cwd = "${workspaceFolder}",
    stopOnEntry = false,       -- (23) 不在入口停下
  },
}
```

---

## 4. Go — Delve

```lua
-- lua/config/dap/go.lua

local dap = require("dap")

dap.adapters.delve = {
  type = "server",
  port = "${port}",
  executable = {
    command = "dlv",
    -- (24) dlv dap 启动 DAP 服务器
    args = { "dap", "-l", "127.0.0.1:${port}" },
  },
}

dap.configurations.go = {
  {
    type = "delve",
    request = "launch",
    name = "Debug file",
    program = "${file}",
  },
  -- (25) 调试测试
  {
    type = "delve",
    request = "launch",
    name = "Debug test",
    mode = "test",
    program = "${workspaceFolder}",
  },
}
```

---

## 5. 关键参考

| 资源 | URL |
|------|-----|
| nvim-dap 文档 | https://github.com/mfussenegger/nvim-dap/blob/master/doc/dap.txt |
| Debug Adapter Wiki | https://codeberg.org/mfussenegger/nvim-dap/wiki/Debug-Adapter-installation |
| nvim-dap-python | https://github.com/mfussenegger/nvim-dap-python |
