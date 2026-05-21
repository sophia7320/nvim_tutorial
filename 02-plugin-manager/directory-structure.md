# 推荐目录结构

> **TL;DR**: 现代化 Neovim 配置采用「配置与插件分离」的模块化结构。核心设置放在 `lua/config/`，插件描述放在 `lua/plugins/`，LSP 配置放在 `lsp/`。

---

## 1. 标准目录结构（2025-2026 社区共识）

```text
~/.config/nvim/
│
├── init.lua                      # 入口文件（最小化）
├── lazy-lock.json                # lazy.nvim 版本锁定文件
│
├── lua/                          # Lua 模块根目录
│   ├── config/                   # 核心配置（非插件）
│   │   ├── options.lua           #   vim.opt 全局选项
│   │   ├── keymaps.lua           #   vim.keymap.set 按键映射
│   │   ├── autocmds.lua          #   vim.api.nvim_create_autocmd
│   │   └── lazy.lua              #   lazy.nvim 引导
│   │
│   └── plugins/                  # 插件描述（lazy.nvim 自动导入）
│       ├── ui.lua                #   主题、状态栏、标签栏、dashboard
│       ├── editor.lua            #   编辑增强（which-key、flash、mini、comment）
│       ├── lsp.lua               #   LSP 客户端 + Mason + conform + nvim-lint
│       ├── completion.lua        #   补全系统（blink.cmp / nvim-cmp + snippets）
│       ├── treesitter.lua        #   Treesitter + textobjects + context
│       ├── telescope.lua         #   Telescope + extensions
│       ├── git.lua               #   gitsigns、neogit、diffview
│       ├── dap.lua               #   调试（nvim-dap + dap-ui）
│       ├── file-explorer.lua     #   neo-tree / oil.nvim
│       ├── ai.lua                #   AI 助手（copilot、codecompanion、avante）
│       └── lang/                 #   语言特定（可选）
│           ├── python.lua
│           ├── rust.lua
│           ├── typescript.lua
│           └── ...
│
├── lsp/                          # LSP 服务器配置（Neovim 0.11+ 原生目录）
│   ├── lua_ls.lua
│   ├── ts_ls.lua
│   └── clangd.lua
│
├── snippets/                     # 自定义代码片段（可选）
│   └── package.json
│
├── after/                        # 后加载覆盖（可选）
│   ├── ftplugin/                 #   文件类型特定设置
│   │   ├── python.lua
│   │   └── markdown.lua
│   └── queries/                  #   Treesitter 查询覆盖
│       └── lua/
│           └── highlights.scm
│
└── .stylua.toml                  # Lua 格式化配置（可选）
```

> 来源：[LazyVim](https://github.com/folke/lazyvim)、[kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim)、[NvChad](https://github.com/NvChad/NvChad)、[fransys.io 2026](https://fransys.io/en/blog/configure-neovim-lua-lazy)、[rubenhortas 2026](https://rubenhortas.github.io/posts/moden-neovim-ide-lua-guide-2026/)

---

## 2. 职责划分原则

| 目录/文件 | 职责 | 示例内容 |
|-----------|------|----------|
| `init.lua` | 入口，**仅 `require` 子模块** | `require("config.options")` |
| `lua/config/options.lua` | 所有 `vim.opt` / `vim.g` 设置 | `vim.opt.number = true` |
| `lua/config/keymaps.lua` | 所有**非 LSP** 的全局按键 | `<leader>ff` → Telescope |
| `lua/config/autocmds.lua` | 所有全局自动命令 | Yank 高亮、回到上次位置 |
| `lua/config/lazy.lua` | lazy.nvim 引导 | 克隆 + `require("lazy").setup()` |
| `lua/plugins/*.lua` | 插件声明与配置 | 返回 `LazySpec` 表 |
| `lsp/*.lua` | LSP 服务器配置 | 返回 `vim.lsp.Config` 表 |
| `after/ftplugin/` | 文件类型特定设置 | `vim.opt_local.tabstop = 4` |

---

## 3. 为什么这样拆分？

### 3.1 配置（config/） vs 插件（plugins/）

```text
lua/config/   → "Neovim 本身的行为怎么调"
lua/plugins/  → "安装和配置哪些插件"
```

**区分理由**：

| 配置 | 插件 |
|------|------|
| 不依赖任何插件 | 依赖插件管理器 |
| 全局生效 | 可能文件类型限定 |
| 先加载 | 后加载（或懒加载） |
| `vim.opt` / `vim.keymap` / `vim.api` | `lazy.nvim` 的 `LazySpec` 格式 |

### 3.2 lsp/ 独立目录（Neovim 0.11+）

```text
lsp/
├── lua_ls.lua  → 返回 { cmd = ..., settings = ... }
└── ts_ls.lua   → 返回 { cmd = ..., settings = ... }
```

**为什么不用 `lua/plugins/` 目录**：
- `lsp/` 是 Neovim 0.11+ 的**原生约定**，Neovim 自动从此目录加载
- 优先级：`lsp/` < `after/lsp/` < `vim.lsp.config()`
- 可与 `nvim-lspconfig` 提供的默认配置合并

---

## 4. 真实仓库结构参考

### LazyVim（folke/lazyvim）

```text
lua/lazyvim/
├── config/
│   ├── init.lua        # 分阶段加载管理器
│   ├── options.lua     # 编辑器选项
│   ├── keymaps.lua     # 延迟到 VeryLazy 后加载
│   └── autocmds.lua    # 延迟到 VeryLazy 后加载
├── plugins/
│   ├── init.lua        # 核心插件
│   ├── ui.lua
│   ├── coding.lua
│   └── ...
└── util/
    ├── init.lua
    └── ...
```

**特色**：keymaps 和 autocmds 延迟加载以优化启动速度。

### kickstart.nvim

**单文件设计**（`init.lua` ≈ 800 行），内部用注释分节：
```lua
-- SECTION 1: FOUNDATION
-- SECTION 2: PLUGIN MANAGER
-- SECTION 3: UI
-- SECTION 4: SEARCH
-- SECTION 5: LSP
-- SECTION 6: FORMATTING
-- SECTION 7: COMPLETION
-- SECTION 8: TREESITTER
```

适合**初学理解全貌**，但不建议作为长期维护方案。

### NvChad

```text
lua/nvchad/
├── options.lua
├── mappings.lua
├── autocmds.lua
├── configs/          # 每个插件独立配置文件
│   ├── cmp.lua
│   ├── lspconfig.lua
│   └── mason.lua
└── plugins/
    └── init.lua      # 插件列表
```

**特色**：`configs/` 目录将插件的复杂配置与插件声明分离。

---

## 5. 文件命名约定

| 文件 | 约定 |
|------|------|
| `lua/config/*.lua` | 小写 + 下划线，如 `keymaps.lua` `lazy.lua` |
| `lua/plugins/*.lua` | 功能描述，如 `ui.lua` `lsp.lua` `git.lua` |
| `lsp/*.lua` | 与 LSP 服务器名一致（`:LspInfo` 中的名称） |
| `after/ftplugin/*.lua` | 与 `:set ft?` 输出一致 |
| `snippets/*.json` | VSCode 格式的片段文件 |

---

## 6. 关键参考

| 资源 | URL |
|------|-----|
| LazyVim 目录结构 | https://github.com/folke/lazyvim |
| kickstart.nvim | https://github.com/nvim-lua/kickstart.nvim |
| NvChad 结构 | https://github.com/NvChad/NvChad |
| lazy.nvim 目录结构指南 | https://lazy.folke.io/usage/structuring |
| 2026 模块化指南 | https://rubenhortas.github.io/posts/moden-neovim-ide-lua-guide-2026/ |
