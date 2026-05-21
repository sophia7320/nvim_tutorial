# Treesitter — 语法树级高亮与文本对象深度解析

> **TL;DR**: Treesitter 将代码解析为 AST（抽象语法树），实现**语义级别**的高亮。本章逐行解析每个配置变量。

---

## 1. 分支选择

| 分支 | 适用 Neovim | tree-sitter CLI | 状态 |
|------|-----------|----------------|------|
| `master` | **0.11 稳定** | 0.20.x | 锁定维护 |
| `main` | 0.12+ nightly | **0.26.1+** | 活跃开发 |

> 本文以 `master` 分支为准（Neovim 0.11）。

---

## 2. 逐行注解：核心配置

```lua
-- lua/plugins/treesitter.lua

return {
  {
    "nvim-treesitter/nvim-treesitter",   -- (1) 插件标识
    branch = "master",                    -- (2) 锁定 0.11 兼容分支
    build = ":TSUpdate",                  -- (3) 更新时自动编译 parser
    event = { "BufReadPre", "BufNewFile" }, -- (4) 打开文件时加载
    -- ╔══════════════════════════════════════════════╗
    -- ║ event 语义详见：                              ║
    -- ║ → 02-plugin-manager/lazy-nvim.md §3.2       ║
    -- ╚══════════════════════════════════════════════╝

    config = function()
      require("nvim-treesitter.configs").setup({
        -- (5) ensure_installed — 自动安装的 parser 列表
        --   每个 parser ~100KB-2MB，全部安装约 ~50MB
        ensure_installed = {
          "c", "lua", "vim", "vimdoc", "query",     -- 基础语言
          "python", "javascript", "typescript",      -- 开发语言
          "rust", "go",
          "html", "css", "json", "yaml", "toml",    -- Web/配置
          "markdown", "markdown_inline", "bash",     -- 文档/脚本
          "regex", "diff",
        },

        auto_install = true,              -- (6) 自动安装缺失 parser
        sync_install = false,             -- (7) 异步安装（不阻塞 UI）

        -- ─── 高亮 ────────────────────────────
        highlight = {
          enable = true,                  -- (8a) 启用 Treesitter 高亮
          additional_vim_regex_highlighting = false, -- (8b) 禁用旧式正则高亮
          -- 注：两套系统同时运行会导致颜色冲突和性能下降
        },

        -- ─── 缩进（实验性）────────────────────
        indent = {
          enable = true,                  -- (9) 基于语法节点的缩进
        },

        -- ─── 增量选择 ────────────────────────
        incremental_selection = {
          enable = true,
          keymaps = {
            init_selection = "<CR>",       -- (10a) 进入增量选择模式
            node_incremental = "<CR>",     -- (10b) 扩大到父节点
            scope_incremental = "<TAB>",   -- (10c) 扩大到作用域
            node_decremental = "<BS>",     -- (10d) 缩小到子节点
          },
        },
      })
    end,
  },
}
```

---

## 3. 文本对象（Textobjects）

```lua
-- lua/plugins/treesitter.lua 续

return {
  {
    "nvim-treesitter/nvim-treesitter-textobjects",
    dependencies = { "nvim-treesitter/nvim-treesitter" },

    config = function()
      require("nvim-treesitter-textobjects").setup({
        -- ─── Select: 语法感知选择 ──────────
        select = {
          enable = true,
          lookahead = true,   -- (11) 光标在节点间隙时智能选取
          keymaps = {
            -- af = "a function"（含 function/end 关键字）
            -- if = "inner function"（仅函数体）
            ["af"] = "@function.outer",
            ["if"] = "@function.inner",
            ["ac"] = "@class.outer",
            ["ic"] = "@class.inner",
            ["aa"] = "@parameter.outer",   -- 含逗号
            ["ia"] = "@parameter.inner",   -- 仅参数名
          },
          selection_modes = {
            ["@function.outer"] = "V",   -- (12) "V" = line-wise（整行更自然）
            ["@parameter.outer"] = "v",  --      "v" = character-wise（默认）
          },
        },

        -- ─── Move: 节点间跳转 ──────────────
        move = {
          enable = true,
          set_jumps = true,     -- (13) 记录跳转历史（可用 <C-o> 返回）
          goto_next_start = {
            ["]f"] = "@function.outer",   -- ]f → 下一函数
            ["]c"] = "@class.outer",      -- ]c → 下一类
          },
          goto_previous_start = {
            ["[f"] = "@function.outer",   -- [f → 上一函数
            ["[c"] = "@class.outer",
          },
        },

        -- ─── Swap: 交换语法节点 ────────────
        swap = {
          enable = true,
          swap_next = { ["<leader>sn"] = "@parameter.inner" },
          swap_previous = { ["<leader>sp"] = "@parameter.inner" },
        },
      })
    end,
  },
}
```

---

## 4. 上下文（Sticky Scroll）

```lua
{
  "nvim-treesitter/nvim-treesitter-context",
  event = "BufReadPre",
  opts = {
    enable = true,                -- (14) 功能开关
    max_lines = 3,                -- (15) 最多显示 3 行
    trim_scope = "outer",         -- (16) 只显示最外层函数名
    mode = "cursor",              -- (17) 根据光标位置更新
  },
}
```

效果：滚动到长函数中间时，窗口顶部始终显示当前函数签名。

---

## 5. 关键参考

| 资源 | URL |
|------|-----|
| nvim-treesitter 文档 | https://nvim-treesitter-nvim-treesitter.mintlify.app/ |
| nvim-treesitter GitHub | https://github.com/nvim-treesitter/nvim-treesitter |
| treesitter-textobjects | https://github.com/nvim-treesitter/nvim-treesitter-textobjects |
| treesitter-context | https://github.com/nvim-treesitter/nvim-treesitter-context |
