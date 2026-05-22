# Treesitter `main` 分支 — Neovim 0.12+ 原生 API 深度解析

> **TL;DR**: `main` 分支是 nvim-treesitter 的完全重写版，仅支持 Neovim 0.12+ nightly。它**放弃**了 `configs.setup()` 一站式配置，改用 Neovim 原生 `vim.treesitter` API。解析器通过 Lua API 显式安装，高亮/缩进/折叠全由 Neovim 原生接管。

---

## 1. 与 `master` 分支的核心差异

| 维度 | `master`（0.11 稳定） | `main`（0.12+ nightly） |
|------|:---:|:---:|
| 配置入口 | `require("nvim-treesitter.configs").setup()` | `require("nvim-treesitter").setup()`（可选） |
| 解析器安装 | `ensure_installed = { ... }` 自动安装 | 显式调用 `require("nvim-treesitter").install({...})` |
| 语法高亮 | `highlight = { enable = true }` | `vim.treesitter.start()` — Neovim 原生控制 |
| 缩进 | `indent = { enable = true }` | `vim.bo.indentexpr` — 在 FileType 事件中手动设置 |
| 折叠 | `fold = { enable = true }` | `vim.treesitter.foldexpr()` — Neovim 原生 |
| 增量选择 | 插件内置 | **Neovim 0.12 内置**，无需插件 |
| 文本对象 | `configs.setup({ textobjects = {...} })` | 独立插件 + **手动绑定键位** |
| build 命令 | `:TSUpdate` / `:TSUpdate stable` | `:TSUpdate`（tier 系统已移除） |
| 懒加载 | 允许（`event = "BufReadPre"`） | **禁止懒加载**（`lazy = false` 必须） |

> ╔══════════════════════════════════════════════╗
> ║ `master` 分支配置详见：                        ║
> ║ → treesitter.md                              ║
> ╚══════════════════════════════════════════════╝

---

## 2. 逐行注解：核心插件配置

```lua
-- ============================================================================
-- lua/plugins/treesitter.lua — nvim-treesitter main 分支
-- 文件：~/.config/nvim/lua/plugins/treesitter.lua
-- 要求：Neovim >= 0.12 nightly + tree-sitter CLI >= 0.26.1
-- ============================================================================

return {
  -- ═══════════════════════════════════════════════════════════════
  -- 组件 1：nvim-treesitter 核心 — 解析器安装器
  -- ═══════════════════════════════════════════════════════════════
  {
    -- (1) 插件标识
    "nvim-treesitter/nvim-treesitter",

    -- (2) lazy = false — 必须立即加载！
    --     main 分支负责注册 runtimepath 中的 parser 和 query 文件
    --     如果懒加载，Neovim 在打开第一个文件时找不到 treesitter 资源
    lazy = false,

    -- (3) build = ":TSUpdate" — 插件更新时自动同步解析器
    --     注意：不再是 :TSUpdate stable！main 分支已移除 tier 系统
    build = ":TSUpdate",

    -- (4) 注意：main 分支**没有** `event` 字段
    --     必须 `lazy = false` 确保启动时立即加载

    config = function()
      -- (5) setup() — 可选，设置全局配置
      --     如果不调用 setup()，一切使用默认值即可
      require("nvim-treesitter").setup({
        -- (5a) install_dir — 解析器的安装目录
        --      默认值：vim.fn.stdpath("data") .. "/site"
        --      显式指定以确保与其他插件一致
        install_dir = vim.fn.stdpath("data") .. "/site",
      })

      -- ─── 6. 安装解析器 ──────────────────────────────
      -- (6) install() — 异步安装指定语言的解析器
      --     参数：语言名列表（与 Neovim 的 filetype 一致）
      --     已安装的解析器不会重复下载，缺失的会自动补装
      --     每个 parser ~100KB-2MB，全部安装约 ~50MB
      require("nvim-treesitter").install({
        -- (6a) Neovim 基础 — 这些必须装
        "lua", "vim", "vimdoc", "query",

        -- (6b) 开发语言
        "python", "javascript", "typescript",
        "rust", "go", "c", "cpp",

        -- (6c) Web / 配置
        "html", "css", "json", "yaml", "toml",

        -- (6d) 文档 / 脚本
        "markdown", "markdown_inline", "bash", "regex", "diff",
      })

      -- (7) 同步安装（仅 bootstrap 脚本使用）
      --     :wait(300000) 最多等 5 分钟
      --     日常配置**不要**用同步安装——会阻塞启动
    end,
  },

  -- ═══════════════════════════════════════════════════════════════
  -- 组件 2：nvim-treesitter-textobjects — 文本对象（独立插件）
  -- ═══════════════════════════════════════════════════════════════
  {
    -- (8) 文本对象插件 — main 分支已完全脱离 configs 体系
    --     键位需要**手动绑定**，不再自动映射
    "nvim-treesitter/nvim-treesitter-textobjects",

    -- (9) branch = "main" — 必须与核心插件一致
    branch = "main",

    -- (10) dependencies — 依赖核心 treesitter 插件
    dependencies = { "nvim-treesitter/nvim-treesitter" },

    -- (11) 延迟到 BufRead 后加载（文本对象只在有文件打开时有用）
    event = "BufReadPost",

    -- (12) init — 在插件加载**前**设置
    --      g:no_plugin_maps = true 禁用所有内置映射
    --      （因为我们要手动绑定，避免冲突）
    init = function()
      vim.g.no_plugin_maps = true
    end,

    config = function()
      -- (13) setup() — 配置 select、move、swap 模块的全局行为
      --      注意：setup() 只负责行为配置，**不会**绑定键位！
      require("nvim-treesitter-textobjects").setup({
        -- ─── Select: 语法感知选择 ──────────
        select = {
          -- (13a) lookahead — 光标在节点间隙时智能选取
          lookahead = true,

          -- (13b) selection_modes — 指定选择模式
          selection_modes = {
            ["@function.outer"] = "V",     -- 函数：整行选择（更自然）
            ["@parameter.outer"] = "v",    -- 参数：字符级选择
          },

          -- (13c) 是否包含包围空白
          include_surrounding_whitespace = false,
        },

        -- ─── Move: 节点间跳转 ──────────────
        move = {
          -- (13d) set_jumps — 记录跳转历史（可用 <C-o> 返回）
          set_jumps = true,
        },

        -- ─── Swap: 交换语法节点 ────────────
        swap = {
          -- (13e) enable — 启用节点交换
          enable = true,
        },
      })

      -- ─── 14. 手动绑定键位 ──────────────────────────
      -- (14) 与 master 分支不同，main 分支**不会自动绑定**
      --      你需要逐个定义按键映射

      local select = require("nvim-treesitter-textobjects.select")

      -- (14a) af / if — a/in 文本对象
      vim.keymap.set({ "x", "o" }, "af", function()
        select.select_textobject("@function.outer", "textobjects")
      end, { desc = "TS: Select outer function" })
      vim.keymap.set({ "x", "o" }, "if", function()
        select.select_textobject("@function.inner", "textobjects")
      end, { desc = "TS: Select inner function" })
      vim.keymap.set({ "x", "o" }, "ac", function()
        select.select_textobject("@class.outer", "textobjects")
      end, { desc = "TS: Select outer class" })
      vim.keymap.set({ "x", "o" }, "ic", function()
        select.select_textobject("@class.inner", "textobjects")
      end, { desc = "TS: Select inner class" })

      -- (14b) Move — 节点间跳转键位
      local move = require("nvim-treesitter-textobjects.move")
      vim.keymap.set({ "n", "x", "o" }, "]f", function()
        move.goto_next_start("@function.outer", "textobjects")
      end, { desc = "TS: Next function" })
      vim.keymap.set({ "n", "x", "o" }, "[f", function()
        move.goto_previous_start("@function.outer", "textobjects")
      end, { desc = "TS: Prev function" })
      vim.keymap.set({ "n", "x", "o" }, "]c", function()
        move.goto_next_start("@class.outer", "textobjects")
      end, { desc = "TS: Next class" })
      vim.keymap.set({ "n", "x", "o" }, "[c", function()
        move.goto_previous_start("@class.outer", "textobjects")
      end, { desc = "TS: Prev class" })

      -- (14c) Swap — 交换参数位置
      local swap = require("nvim-treesitter-textobjects.swap")
      vim.keymap.set("n", "<leader>sn", function()
        swap.swap_next("@parameter.inner", "textobjects")
      end, { desc = "TS: Swap next param" })
      vim.keymap.set("n", "<leader>sp", function()
        swap.swap_previous("@parameter.inner", "textobjects")
      end, { desc = "TS: Swap prev param" })
    end,
  },

  -- ═══════════════════════════════════════════════════════════════
  -- 组件 3：nvim-treesitter-context — 上下文显示（不变）
  -- ═══════════════════════════════════════════════════════════════
  {
    -- (15) treesitter-context 不依赖 configs 体系，配置不变
    "nvim-treesitter/nvim-treesitter-context",

    -- (16) 事件驱动延迟加载
    event = "BufReadPost",

    -- (17) 依赖核心 treesitter 插件
    dependencies = { "nvim-treesitter/nvim-treesitter" },

    opts = {
      -- (17a) enable — 功能开关
      enable = true,

      -- (17b) max_lines — 最多显示几行上下文
      max_lines = 3,

      -- (17c) trim_scope — 只显示最外层函数/类名
      trim_scope = "outer",

      -- (17d) mode — "cursor" 根据光标位置更新上下文
      mode = "cursor",
    },
  },
}
```

---

## 3. 逐行注解：启用 Neovim 原生 Treesitter 功能

以下代码放在 `lua/config/autocmds.lua` 或 `lua/config/options.lua` 中：

```lua
-- ============================================================================
-- Neovim 原生 Treesitter 功能 — 替代 master 分支的 highlight/indent/fold
-- ============================================================================

-- (1) FileType autocmd — 打开文件时自动启用 treesitter 功能
vim.api.nvim_create_autocmd("FileType", {
  -- (2) group — 使用 augroup 防止重复注册
  group = vim.api.nvim_create_augroup("my_treesitter", { clear = true }),

  -- (3) pattern — 限定启用的文件类型
  pattern = {
    "lua", "vim", "python", "javascript", "typescript",
    "rust", "go", "c", "cpp", "markdown",
    "html", "css", "json", "yaml", "toml", "bash",
  },

  callback = function(args)
    -- (4) pcall 保护 — 如果该语言的 parser 未安装，静默跳过
    local ok, _ = pcall(vim.treesitter.language.add, args.match)
    if not ok then return end

    -- (5) vim.treesitter.start() — 启用语法高亮
    --     这是 main 分支的 "highlight = { enable = true }" 等价
    vim.treesitter.start(args.buf, args.match)

    -- (6) 折叠 — 使用 treesitter 语法节点判断折叠
    --     等价于 master 分支的 fold = { enable = true }
    vim.wo.foldexpr = "v:lua.vim.treesitter.foldexpr()"
    vim.wo.foldmethod = "expr"

    -- (7) 缩进 — 使用 treesitter 语法感知缩进
    --     等价于 master 分支的 indent = { enable = true }
    vim.bo[args.buf].indentexpr = "v:lua.require'nvim-treesitter'.indentexpr()"
  end,
})
```

**效果对照**：

| 功能 | master 分支写法 | main 分支等价（Neovim 原生） |
|------|------|------|
| 语法高亮 | `highlight = { enable = true }` | `vim.treesitter.start(buf, lang)` |
| 缩进 | `indent = { enable = true }` | `vim.bo.indentexpr = "v:lua.require'nvim-treesitter'.indentexpr()"` |
| 折叠 | `fold = { enable = true }` | `vim.wo.foldexpr = "v:lua.vim.treesitter.foldexpr()"` |
| 增量选择 | `incremental_selection = { enable = true }` | **Neovim 0.12 内置**，`vim.treesitter.incremental_selection()` |

---

## 4. master → main 迁移速查表

| 旧版 (`master`) | 新版 (`main`) | 说明 |
|---|---|---|
| `require("nvim-treesitter.configs").setup({...})` | `require("nvim-treesitter").setup({...})`（可选） | 不再需要 `configs` 子模块 |
| `ensure_installed = { "lua", ... }` | `require("nvim-treesitter").install({"lua", ...})` | 显式 Lua API 调用 |
| `auto_install = true` | **已移除** — `install()` 会自动跳过已安装项 | 不再需要该字段 |
| `sync_install = false` | 异步是唯一方式（除非用 `:wait()`） | |
| `highlight = { enable = true }` | `vim.treesitter.start()` | Neovim 原生 API |
| `indent = { enable = true }` | `vim.bo.indentexpr` | 在 FileType 中手动设置 |
| `incremental_selection` | **Neovim 0.12 内置** `vim.treesitter.incremental_selection()` | 无需插件 |
| `textobjects.select/move/swap` | 独立插件 + 手动绑定键位 | API 签名变化 |
| `build = ":TSUpdate stable"` | `build = ":TSUpdate"` | tier 系统已移除 |
| `lazy = false`（或事件驱动） | **`lazy = false` 必须** | main 分支不能懒加载 |

---

## 5. 系统依赖

```bash
# tree-sitter CLI >= 0.26.1（编译 parser 需要）
cargo install tree-sitter-cli

# 或通过系统包管理器（推荐）
# macOS: brew install tree-sitter
# Arch:  pacman -S tree-sitter

# 验证版本
tree-sitter --version   # 应输出 >= 0.26.1
```

---

## 6. 关键参考

| 资源 | URL |
|------|-----|
| nvim-treesitter main README | https://github.com/nvim-treesitter/nvim-treesitter/blob/main/README.md |
| nvim-treesitter-textobjects main | https://github.com/nvim-treesitter/nvim-treesitter-textobjects |
| nvim-treesitter-context | https://github.com/nvim-treesitter/nvim-treesitter-context |
| main 分支迁移讨论 | https://github.com/nvim-treesitter/nvim-treesitter/issues/7926 |
| master 分支配置（本教程） | [treesitter.md](treesitter.md) |
