# lazy.nvim — 现代插件管理器深度解析

> **TL;DR**: lazy.nvim 是 2024-2026 年 Neovim 生态的**唯一标准**插件管理器。默认懒加载、支持版本锁定、提供可视化 UI、自动编译缓存。

---

## 1. 为什么 lazy.nvim 是唯一选择？

| 对比维度 | lazy.nvim | packer.nvim | vim-plug |
|----------|-----------|-------------|----------|
| 状态 | **活跃维护** | ⚠️ 已归档 | 维护模式 |
| 语言 | 纯 Lua | Lua | Vimscript |
| 懒加载 | **默认开启** | 需手动配置 | 需手动配置 |
| UI 界面 | **内置** `:Lazy` | 无 | 无 |
| 版本锁定 | `lazy-lock.json` | `snap` | 无 |
| 启动速度 | ~11ms (93 plugins) | ~30ms | ~80ms |
| 配置合并 | `opts` 自动合并 | 手动 | 无 |
| import 目录 | **自动发现** | 需 `require` | 无 |

> 来源：[lazy.nvim 官方文档](https://lazy.folke.io/) | [folke 的 93 插件 11ms 启动](https://lazy.folke.io/usage/profiling)

---

## 2. 安装与引导

```lua
-- ~/.config/nvim/lua/config/lazy.lua
-- ============================================================================
-- lazy.nvim 引导脚本
-- 首次运行时自动克隆安装
-- ============================================================================

local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not (vim.uv or vim.loop).fs_stat(lazypath) then
  local lazyrepo = "https://github.com/folke/lazy.nvim.git"
  local out = vim.fn.system({
    "git", "clone",
    "--filter=blob:none",  -- partial clone（不下载历史 blob）
    "--branch=stable",
    lazyrepo, lazypath,
  })
  if vim.v.shell_error ~= 0 then
    vim.api.nvim_echo({
      { "Failed to clone lazy.nvim:\n", "ErrorMsg" },
      { out, "WarningMsg" },
      { "\nPress any key to exit..." },
    }, true, {})
    vim.fn.getchar()
    os.exit(1)
  end
end
vim.opt.rtp:prepend(lazypath)

-- 调用 lazy.setup
require("lazy").setup({
  spec = {
    { import = "plugins" }, -- 自动加载 lua/plugins/ 下所有 .lua 文件
  },
  defaults = {
    lazy = true,            -- 默认启用懒加载
    version = false,        -- 默认使用最新 commit（而非 Semver）
  },
  install = {
    colorscheme = { "catppuccin" }, -- 安装期间使用的主题
  },
  checker = {
    enabled = true,         -- 自动检查更新
    notify = false,         -- 不弹通知
  },
  change_detection = {
    notify = false,
  },
  performance = {
    cache = { enabled = true },
    rtp = {
      reset = true,         -- 重置 runtime path
      disabled_plugins = {  -- 禁用内置插件
        "gzip",
        "netrwPlugin",
        "tarPlugin",
        "tutor",
        "zipPlugin",
      },
    },
  },
  ui = {
    border = "rounded",     -- 圆角边框
  },
})
```

> 来源：[lazy.nvim 安装指南](https://lazy.folke.io/installation)

---

## 3. Plugin Spec（插件描述）完整参考

### 3.1 来源定义

```lua
-- 短 URL（GitHub）
{ "folke/tokyonight.nvim" }

-- 完整 URL
{ "https://github.com/folke/tokyonight.nvim" }

-- 本地目录
{ dir = "~/projects/my-plugin" }

-- 自定义名称
{ "folke/tokyonight.nvim", name = "tokyonight" }
```

### 3.2 加载控制

```lua
-- 事件驱动：在 BufReadPre/BufNewFile 后加载
{ "nvim-treesitter/nvim-treesitter", event = { "BufReadPre", "BufNewFile" } }

-- 命令驱动：执行 :StartupTime 时加载
{ "dstein64/vim-startuptime", cmd = "StartupTime" }

-- 文件类型驱动：打开 .norg 文件时加载
{ "nvim-neorg/neorg", ft = "norg" }

-- 按键驱动：按下 <leader>e 时加载
{
  "nvim-neo-tree/neo-tree.nvim",
  keys = { { "<leader>e", "<cmd>Neotree toggle<CR>", desc = "文件浏览器" } },
}

-- 条件加载：不在 VS Code 中加载
{ "some/plugin", cond = not vim.g.vscode }

-- VeryLazy：所有非延迟插件加载完成后
{ "stevearc/dressing.nvim", event = "VeryLazy" }

-- 立即加载（不使用懒加载）
{ "folke/tokyonight.nvim", lazy = false, priority = 1000 }
```

### 3.3 配置方式

```lua
-- ✅ 推荐：使用 opts（自动调用 require("plugin").setup(opts)）
{ "folke/todo-comments.nvim", opts = { signs = false } }

-- ✅ 需要自定义时使用 config
{
  "folke/tokyonight.nvim",
  lazy = false,
  priority = 1000,
  config = function()
    require("tokyonight").setup({ style = "storm" })
    vim.cmd.colorscheme("tokyonight")
  end,
}

-- ✅ 启动前设置：使用 init
{
  "dstein64/vim-startuptime",
  cmd = "StartupTime",
  init = function()
    vim.g.startuptime_tries = 10
  end,
}

-- ❌ 不推荐：config 仅调用 setup（用 opts 代替）
{ "plugin/nvim", config = function() require("plugin").setup({}) end }
```

### 3.4 版本控制

```lua
-- 固定分支
{ "folke/tokyonight.nvim", branch = "main" }

-- 固定标签
{ "nvim-telescope/telescope.nvim", tag = "0.1.8" }

-- Semver 范围
{ "saghen/blink.cmp", version = "1.*" }

-- 固定 commit（不推荐，维护麻烦）
{ "folke/tokyonight.nvim", commit = "abc123" }

-- 锁定不更新
{ "nvim-lua/plenary.nvim", pin = true }
```

### 3.5 依赖管理

```lua
-- 依赖自动延迟加载
{
  "nvim-telescope/telescope.nvim",
  dependencies = {
    "nvim-lua/plenary.nvim",  -- 自动在 Telescope 加载时加载
    { "nvim-telescope/telescope-fzf-native.nvim", build = "make" },
  },
}

-- 纯 Lua 库不需要放在 dependencies 中
-- require("plenary") 会自动加载
```

> 来源：[lazy.nvim spec 参考](https://lazy.folke.io/spec)

---

## 4. 目录结构：plugins/ 自动导入

lazy.nvim 的 `import = "plugins"` 会自动扫描 `lua/plugins/` 下所有 `.lua` 文件：

```text
~/.config/nvim/
└── lua/
    └── plugins/
        ├── ui.lua            -- 主题、状态栏、图标
        ├── editor.lua        -- 编辑增强（autopairs、comment、which-key、flash）
        ├── lsp.lua           -- LSP 相关（mason、lspconfig、conform、lint）
        ├── completion.lua    -- 补全（blink.cmp / nvim-cmp + snippets）
        ├── treesitter.lua    -- Treesitter + textobjects + context
        ├── telescope.lua     -- Telescope + extensions
        ├── git.lua           -- gitsigns、neogit、diffview
        ├── dap.lua           -- nvim-dap + dap-ui
        ├── ai.lua            -- copilot / codecompanion / avante
        └── lang/             -- 语言特定
            ├── python.lua
            ├── rust.lua
            └── typescript.lua
```

每个文件返回一个 plugin spec 或 spec 列表（table），lazy.nvim 自动合并。

```lua
-- lua/plugins/ui.lua
return {
  {
    "folke/tokyonight.nvim",
    lazy = false,
    priority = 1000,
    opts = { style = "storm" },
  },
  {
    "nvim-lualine/lualine.nvim",
    event = "VeryLazy",
    opts = { theme = "tokyonight" },
  },
  {
    "nvim-tree/nvim-web-devicons",
    lazy = true, -- 作为依赖自动加载
  },
}
```

---

## 5. 延迟加载策略最佳实践

### 5.1 哪些不应该懒加载？

```lua
-- 配色方案：必须最先加载
{ "folke/tokyonight.nvim", lazy = false, priority = 1000 }

-- Treesitter（新版 main 分支不支持懒加载）
{ "nvim-treesitter/nvim-treesitter", lazy = false, build = ":TSUpdate" }
```

### 5.2 哪些用 event 加载？

```lua
-- 编辑增强：UI 渲染完成后
{ "folke/which-key.nvim", event = "VeryLazy" }
{ "echasnovski/mini.nvim", event = "VeryLazy" }
{ "stevearc/dressing.nvim", event = "VeryLazy" }

-- 文件操作：在打开文件时
{ "nvim-treesitter/nvim-treesitter-context", event = "BufReadPre" }

-- 终端：打开终端时
{ "akinsho/toggleterm.nvim", event = "TermOpen" }
```

### 5.3 哪些用 keys 加载？

```lua
{
  "folke/flash.nvim",
  keys = {
    { "s", mode = { "n", "x", "o" }, function() require("flash").jump() end, desc = "Flash" },
  },
}
```

---

## 6. 常用命令

| 命令 | 说明 |
|------|------|
| `:Lazy` | 打开插件管理面板（查看状态、更新、日志、性能分析） |
| `:Lazy sync` | 安装缺失、更新、删除未使用插件 |
| `:Lazy update` | 更新所有插件 |
| `:Lazy clean` | 删除不在配置中的插件 |
| `:Lazy restore` | 根据 `lazy-lock.json` 恢复插件版本 |
| `:Lazy check` | 检查插件健康状态 |
| `:Lazy profile` | 启动性能分析 |

---

## 7. lazy-lock.json（版本锁定）

`lazy-lock.json` 记录所有插件的精确 commit SHA：

```json
{
  "lazy.nvim": { "branch": "main", "commit": "abc123..." },
  "tokyonight.nvim": { "branch": "main", "commit": "def456..." },
  ...
}
```

**最佳实践**：
- **将 `lazy-lock.json` 加入 Git 版本控制**
- 在不同机器间 `:Lazy restore` 恢复相同版本
- 不要手动编辑

---

## 8. 迁移指南

### 从 packer.nvim 迁移

| packer.nvim | lazy.nvim |
|-------------|-----------|
| `use 'folke/tokyonight.nvim'` | `{ "folke/tokyonight.nvim" }` |
| `use { 'plugin/nvim', config = function() ... end }` | `{ "plugin/nvim", config = function() ... end }` |
| `setup` | `init` |
| `requires` | `dependencies` |
| `as` | `name` |
| `opt = true` | `lazy = true`（已是默认） |
| `run` | `build` |
| `lock = true` | `pin = true` |
| `disable = true` | `enabled = false` |
| `tag = '*'` | `version = "*"` |
| `after` / `wants` | `dependencies`（大多数情况不需要） |

### 从 vim-plug 迁移

```vim
" ❌ 旧
Plug 'folke/tokyonight.nvim'

" ✅ 新
{ "folke/tokyonight.nvim" }
```

> 来源：[lazy.nvim 迁移指南](https://lazy.folke.io/usage/migration)

---

## 9. 关键参考

| 资源 | URL |
|------|-----|
| lazy.nvim 官方文档 | https://lazy.folke.io/ |
| Plugin Spec 完整参考 | https://lazy.folke.io/spec |
| 延迟加载详解 | https://lazy.folke.io/spec/lazy_loading |
| 目录结构指南 | https://lazy.folke.io/usage/structuring |
| 性能分析指南 | https://lazy.folke.io/usage/profiling |
| 迁移指南 | https://lazy.folke.io/usage/migration |
| GitHub 仓库 | https://github.com/folke/lazy.nvim |
| LazyVim 配置示例 | https://github.com/LazyVim/LazyVim/tree/main/lua/lazyvim/plugins |
