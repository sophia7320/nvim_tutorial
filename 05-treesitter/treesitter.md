# Treesitter — 语法树级高亮与代码分析

> **TL;DR**: Treesitter 是 Neovim 的语法解析引擎，提供**基于 AST 的高亮**（非正则）、语法感知文本对象、缩进、折叠等功能。它是现代 Neovim 配置的必备组件。

---

## 1. 重要版本说明

nvim-treesitter 正在进行一次彻底重写：

| 分支 | 目标 Neovim | 依赖 | 状态 |
|------|------------|------|------|
| `master` | 0.11（稳定） | tree-sitter CLI 0.20.x | 锁定维护，不再添加新功能 |
| `main` | 0.12+（nightly） | tree-sitter-cli 0.26.1+ | 活跃开发，新范式 |

> 来源：[nvim-treesitter 官方文档](https://nvim-treesitter-nvim-treesitter.mintlify.app/guides/setup)

---

## 2. 安装与基础配置

### 2.1 master 分支（Neovim 0.11 稳定推荐）

```lua
-- lua/plugins/treesitter.lua
return {
  {
    "nvim-treesitter/nvim-treesitter",
    branch = "master", -- 0.11 稳定兼容
    build = ":TSUpdate",
    event = { "BufReadPre", "BufNewFile" },
    config = function()
      require("nvim-treesitter.configs").setup({
        -- 自动安装的 parser 列表
        ensure_installed = {
          "c", "lua", "vim", "vimdoc", "query",
          "python", "javascript", "typescript", "rust", "go",
          "html", "css", "json", "yaml", "toml",
          "markdown", "markdown_inline",
        },
        -- 自动安装缺失的 parser
        auto_install = true,
        -- 语法高亮
        highlight = {
          enable = true,
          -- 禁用 vim 正则高亮（避免冲突和性能下降）
          additional_vim_regex_highlighting = false,
        },
        -- 缩进（实验性）
        indent = { enable = true },
        -- 增量选择
        incremental_selection = {
          enable = true,
          keymaps = {
            init_selection = "<CR>",     -- 开始选择
            node_incremental = "<CR>",   -- 扩大节点
            scope_incremental = "<TAB>", -- 扩大作用域
            node_decremental = "<BS>",   -- 缩小节点
          },
        },
      })
    end,
  },

  -- 上下文（sticky scroll）
  {
    "nvim-treesitter/nvim-treesitter-context",
    event = "BufReadPre",
    opts = {
      max_lines = 3,
      trim_scope = "outer",
    },
  },
}
```

### 2.2 main 分支（Neovim 0.12+ nightly，未来方向）

```lua
{
  "nvim-treesitter/nvim-treesitter",
  branch = "main",
  lazy = false,        -- ❗️ 新版不支持懒加载
  build = ":TSUpdate",
  config = function()
    require("nvim-treesitter").setup({
      install_dir = vim.fn.stdpath("data") .. "/treesitter",
    })
    -- 安装 parser
    require("nvim-treesitter").install({
      "lua", "python", "javascript", "typescript", "rust", "go",
    })
  end,
}
```

**新版范式变化**：
- 不再使用 `configs.setup()` 模块系统
- 高亮通过 `vim.treesitter.start()` 在 FileType autocmd 中启用
- `incremental_selection` 模块已移至 `nvim-treesitter-textobjects`

---

## 3. 语法文本对象（Textobjects）

`nvim-treesitter-textobjects` 提供语法感知的文本对象和移动操作：

```lua
{
  "nvim-treesitter/nvim-treesitter-textobjects",
  branch = "main",
  dependencies = { "nvim-treesitter/nvim-treesitter" },
  config = function()
    require("nvim-treesitter-textobjects").setup({
      -- 选择（Select）
      select = {
        enable = true,
        lookahead = true,
        keymaps = {
          ["af"] = "@function.outer",
          ["if"] = "@function.inner",
          ["ac"] = "@class.outer",
          ["ic"] = "@class.inner",
          ["al"] = "@loop.outer",
          ["il"] = "@loop.inner",
          ["aa"] = "@parameter.outer",  -- 函数参数
          ["ia"] = "@parameter.inner",
        },
        selection_modes = {
          ["@parameter.outer"] = "v",
          ["@function.outer"] = "V",
        },
      },

      -- 移动（Move）
      move = {
        enable = true,
        set_jumps = true, -- 记录跳转历史
        goto_next_start = {
          ["]f"] = "@function.outer",
          ["]c"] = "@class.outer",
        },
        goto_previous_start = {
          ["[f"] = "@function.outer",
          ["[c"] = "@class.outer",
        },
      },

      -- 交换（Swap）
      swap = {
        enable = true,
        swap_next = {
          ["<leader>sn"] = "@parameter.inner",
        },
        swap_previous = {
          ["<leader>sp"] = "@parameter.inner",
        },
      },
    })
  end,
}
```

**常用操作**：
- `vaf` → 选中整个函数
- `daf` → 删除整个函数
- `]f` → 跳到下一个函数
- `[f` → 跳到上一个函数
- `<leader>sn` / `<leader>sp` → 交换函数参数顺序

> 来源：[nvim-treesitter-textobjects README](https://github.com/nvim-treesitter/nvim-treesitter-textobjects)

---

## 4. 上下文（Sticky Scroll）

`nvim-treesitter-context` 在窗口顶部显示当前作用域（函数名/类名）：

```lua
{
  "nvim-treesitter/nvim-treesitter-context",
  event = "BufReadPre",
  opts = {
    enable = true,
    max_lines = 3,            -- 最多显示 3 行
    min_window_height = 20,   -- 窗口太矮时不显示
    line_numbers = true,
    multiline_threshold = 20,
    trim_scope = "outer",
    mode = "cursor",          -- cursor 或 top_line
    separator = nil,          -- 分隔线样式
    zindex = 20,
  },
}
```

效果类似 VS Code 的 sticky scroll——当你滚动到一个长函数的中间时，顶部始终显示当前所在函数的签名。

---

## 5. 彩虹括号（Rainbow Delimiters）

```lua
{
  "HiPhish/rainbow-delimiters.nvim",
  event = "BufReadPre",
}
```

---

## 6. Treesitter 核心用途速查

| 用途 | 方式 | 依赖 |
|------|------|------|
| 语法高亮 | nvim-treesitter `highlight.enable = true` | 核心 |
| 代码折叠 | `foldmethod=expr, foldexpr=v:lua.vim.treesitter.foldexpr()` | 核心 |
| 缩进 | nvim-treesitter `indent.enable = true` | 核心（实验性） |
| 文本对象 | nvim-treesitter-textobjects | 独立插件 |
| 上下文 | nvim-treesitter-context | 独立插件 |
| 彩虹括号 | rainbow-delimiters.nvim | 独立插件 |
| 语法节点跳转 | flash.nvim treesitter 模式 | flash.nvim |
| 注释感知 | ts-comments.nvim (folke) | 独立插件 |

---

## 7. 故障排查

### Treesitter parser 编译失败

```bash
# 确保安装了 C 编译器
sudo apt install build-essential

# 确保安装了 tree-sitter CLI
npm install -g tree-sitter-cli

# 手动重新安装 parser
:TSInstallSync lua
```

### 高亮不生效

```vim
:checkhealth nvim-treesitter
:TSInstallInfo   " 检查 parser 安装状态
```

---

## 8. 关键参考

| 资源 | URL |
|------|-----|
| nvim-treesitter 官方文档 | https://nvim-treesitter-nvim-treesitter.mintlify.app/ |
| nvim-treesitter GitHub | https://github.com/nvim-treesitter/nvim-treesitter |
| treesitter-textobjects | https://github.com/nvim-treesitter/nvim-treesitter-textobjects |
| treesitter-context | https://github.com/nvim-treesitter/nvim-treesitter-context |
| rainbow-delimiters | https://github.com/HiPhish/rainbow-delimiters.nvim |
