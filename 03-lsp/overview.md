# LSP 生态全景 — Neovim 0.11+ 语言服务器配置

> **TL;DR**: Neovim 0.11 引入了原生 `vim.lsp.config()` + `vim.lsp.enable()` API，彻底改变了 LSP 配置方式。Mason 安装工具 → LSP 客户端配置 → Conform 格式化 → nvim-lint 代码检查，形成完整的开发环境管道。

---

## 1. 架构全景图

```
┌─────────────────────────────────────────────────────┐
│                    Neovim 编辑器                      │
├─────────────────────────────────────────────────────┤
│  vim.lsp.config() / vim.lsp.enable()  ← 原生 API    │
│  vim.diagnostic.config()               ← 诊断显示    │
│  vim.lsp.completion                     ← 原生补全    │
├─────────────────────────────────────────────────────┤
│                                                      │
│  ┌──────────────┐   ┌──────────────┐                │
│  │ mason.nvim   │   │nvim-lspconfig│                │
│  │ (工具安装器) │   │ (配置数据集) │                │
│  └──────┬───────┘   └──────┬───────┘                │
│         │                   │                        │
│  ┌──────▼───────────────────▼───────┐                │
│  │     mason-lspconfig.nvim         │                │
│  │     (桥接层：名称映射+自动激活)    │                │
│  └──────────────────────────────────┘                │
│                                                      │
│  ┌──────────────┐   ┌──────────────┐                │
│  │conform.nvim  │   │  nvim-lint   │                │
│  │  (格式化)    │   │  (代码检查)  │                │
│  └──────────────┘   └──────────────┘                │
│                                                      │
│  ┌──────────────────────────────────┐                │
│  │  LSP 服务器 (外部进程)            │                │
│  │  lua_ls / ts_ls / rust_analyzer  │                │
│  │  pyright / gopls / clangd ...    │                │
│  └──────────────────────────────────┘                │
└─────────────────────────────────────────────────────┘
```

---

## 2. Neovim 0.11+ 范式转变

### 旧范式（0.10 及以前）

```lua
-- ❌ 已废弃
local lspconfig = require("lspconfig")
lspconfig.lua_ls.setup({
  settings = { ... }
})
```

### 新范式（0.11+）

```lua
-- ✅ 现代方式
vim.lsp.config("lua_ls", {
  settings = {
    Lua = { ... }
  },
})
vim.lsp.enable("lua_ls")
```

### nvim-lspconfig 的角色变化

**nvim-lspconfig 本身并未废弃**，但角色发生了根本变化：

| 以前 | 现在 |
|------|------|
| LSP 客户端框架 | **配置数据集合** |
| `require("lspconfig").xxx.setup()` | 提供 `lsp/` 目录下的预设配置 |
| 必须显式调用 setup | `vim.lsp.enable()` 自动加载 |

> 来源：[nvim-lspconfig README](https://github.com/neovim/nvim-lspconfig/blob/master/README.md)

---

## 3. 配置优先级

Neovim 0.11+ 中，LSP 配置有严格的优先级顺序（从低到高）：

```
1. nvim-lspconfig 提供的 lsp/*.lua 默认配置  ← 最低
2. 用户 ~/.config/nvim/lsp/ 目录下的覆盖文件
3. 用户 ~/.config/nvim/after/lsp/ 目录下的覆盖文件
4. vim.lsp.config() 运行时调用                    ← 最高
```

这意味着：
- 你可以在 `lsp/lua_ls.lua` 中放置持久化配置
- 也可以在 `vim.lsp.config()` 中进行动态覆盖

---

## 4. 核心插件生态与职责分工

| 插件 | 职责 | 装载方式 |
|------|------|----------|
| **mason.nvim** | 安装/管理外部工具（LSP、DAP、Linter、Formatter） | 不懒加载 |
| **nvim-lspconfig** | 提供 200+ LSP 服务器的预设配置 | 依赖 mason |
| **mason-lspconfig.nvim** | 桥接：名称映射 + `ensure_installed` + 自动激活 | 依赖以上两者 |
| **conform.nvim** | 异步格式化（替代 LSP formatting） | `event = "BufWritePre"` |
| **nvim-lint** | 异步代码检查，注入 `vim.diagnostic` | `event = "BufReadPre"` |

---

## 5. 快速开始：最小可用 LSP 配置

```lua
-- ~/.config/nvim/lua/plugins/lsp.lua
return {
  -- 1. Mason：工具安装器（不应懒加载）
  {
    "mason-org/mason.nvim",
    opts = {
      ui = {
        icons = {
          package_installed = "✓",
          package_pending = "➜",
          package_uninstalled = "✗",
        },
      },
    },
  },

  -- 2. nvim-lspconfig：配置集合
  {
    "neovim/nvim-lspconfig",
    dependencies = { "mason-org/mason.nvim" },
  },

  -- 3. mason-lspconfig：桥接层
  {
    "mason-org/mason-lspconfig.nvim",
    dependencies = {
      "mason-org/mason.nvim",
      "neovim/nvim-lspconfig",
    },
    opts = {
      ensure_installed = { "lua_ls", "ts_ls", "rust_analyzer", "pyright" },
      automatic_enable = true, -- 自动 vim.lsp.enable()
    },
  },

  -- 4. 格式化
  {
    "stevearc/conform.nvim",
    event = "BufWritePre",
    opts = {
      formatters_by_ft = {
        lua = { "stylua" },
        python = { "ruff_format" },
        javascript = { "prettierd", "prettier", stop_after_first = true },
        typescript = { "prettierd", "prettier", stop_after_first = true },
        go = { "goimports", "gofmt" },
        rust = { "rustfmt" },
      },
      format_on_save = { timeout_ms = 500, lsp_format = "fallback" },
    },
  },

  -- 5. 代码检查
  {
    "mfussenegger/nvim-lint",
    event = { "BufReadPre", "BufNewFile" },
    config = function()
      require("lint").linters_by_ft = {
        python = { "ruff" },
        javascript = { "eslint_d" },
        typescript = { "eslint_d" },
        lua = { "luacheck" },
      }
      vim.api.nvim_create_autocmd("BufWritePost", {
        callback = function() require("lint").try_lint() end,
      })
    end,
  },
}
```

**配合的 LSP 配置**：

```lua
-- ~/.config/nvim/lsp/lua_ls.lua
---@type vim.lsp.Config
return {
  cmd = { "lua-language-server" },
  filetypes = { "lua" },
  root_markers = { ".luarc.json", ".luarc.jsonc", ".git" },
  settings = {
    Lua = {
      runtime = { version = "LuaJIT" },
      diagnostics = { globals = { "vim" } },
      workspace = { checkThirdParty = false, library = { vim.env.VIMRUNTIME } },
      telemetry = { enable = false },
    },
  },
}
```

---

## 6. 诊断配置

```lua
-- ~/.config/nvim/lua/config/lsp.lua
vim.diagnostic.config({
  virtual_text = {
    source = "if_many",  -- 多来源时显示来源
    prefix = "●",
    spacing = 4,
  },
  signs = {
    text = {
      [vim.diagnostic.severity.ERROR] = " ",
      [vim.diagnostic.severity.WARN] = " ",
      [vim.diagnostic.severity.INFO] = " ",
      [vim.diagnostic.severity.HINT] = "󰌵 ",
    },
  },
  underline = true,
  update_in_insert = false,
  severity_sort = true,
  float = { border = "rounded", source = true },
})
```

---

## 7. Neovim 0.11 内置默认 LSP Keymaps

Neovim 0.11 起**自动创建**以下 LSP 映射，无需手动设置：

| Keymap | 模式 | 功能 |
|--------|------|------|
| `grn` | Normal | 重命名 (`vim.lsp.buf.rename()`) |
| `gra` | Normal/Visual | 代码操作 (`vim.lsp.buf.code_action()`) |
| `grr` | Normal | 查找引用 (`vim.lsp.buf.references()`) |
| `gri` | Normal | 查找实现 (`vim.lsp.buf.implementation()`) |
| `grt` | Normal | 类型定义 (`vim.lsp.buf.type_definition()`) |
| `grx` | Normal | 执行 CodeLens (`vim.lsp.codelens.run()`) |
| `gO` | Normal | 文档符号 (`vim.lsp.buf.document_symbol()`) |
| `<C-S>` | Insert | 签名帮助 (`vim.lsp.buf.signature_help()`) |
| `K` | Normal | 悬停文档（LSP 激活时自动设置） |
| `gd` | Normal | 跳转到定义（通过 tagfunc） |

---

## 8. 关键参考

| 资源 | URL |
|------|-----|
| Neovim 官方 LSP 文档 | https://neovim.io/doc/user/lsp/ |
| Neovim 诊断文档 | https://neovim.io/doc/user/diagnostic/ |
| nvim-lspconfig | https://github.com/neovim/nvim-lspconfig |
| mason.nvim | https://github.com/mason-org/mason.nvim |
| mason-lspconfig.nvim | https://github.com/mason-org/mason-lspconfig.nvim |
| conform.nvim | https://github.com/stevearc/conform.nvim |
| nvim-lint | https://github.com/mfussenegger/nvim-lint |
| Neovim 0.11 新闻 | https://neovim.io/doc/user/news-0.11.html |
| Native LSP guide | https://dotfiles.substack.com/p/native-lsp-in-neovim-012 |
