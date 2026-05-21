# mini.nvim — 轻量级工具套件

> **TL;DR**: mini.nvim 是一组 40+ 独立 Lua 模块，每个模块可单独使用。可替代多个单功能插件（autopairs、surround、comment、files、pick 等）。

---

## 1. 推荐启用模块

| 模块 | 功能 | 替代品 |
|------|------|--------|
| **mini.pairs** | 自动配对括号/引号 | nvim-autopairs |
| **mini.surround** | 添加/删除/替换包围字符 | nvim-surround |
| **mini.ai** | 语法感知文本对象 | (额外增强) |
| **mini.comment** | 行/块注释 | Comment.nvim |
| **mini.files** | 列式文件浏览器 | neo-tree / oil.nvim |
| **mini.pick** | 轻量模糊选择器 | Telescope |
| **mini.statusline** | 极简状态栏 | lualine.nvim |
| **mini.tabline** | 极简缓冲区标签 | bufferline.nvim |
| **mini.icons** | 文件/UI 图标 | nvim-web-devicons |
| **mini.indentscope** | 缩进作用域动画 | indent-blankline.nvim |
| **mini.diff** | 缓冲区内 diff | (额外) |
| **mini.snippets** | 代码片段引擎 | LuaSnip |
| **mini.clue** | 按键提示 | which-key.nvim |

---

## 2. 安装与配置

```lua
-- lua/plugins/editor.lua
{
  "nvim-mini/mini.nvim",
  event = "VeryLazy",
  config = function()
    -- 自动配对
    require("mini.pairs").setup({
      modes = { insert = true, command = false, terminal = false },
    })

    -- 包围操作（sa 添加, sd 删除, sr 替换）
    require("mini.surround").setup({
      mappings = {
        add = "sa",
        delete = "sd",
        replace = "sr",
      },
    })

    -- 增强文本对象（支持 Treesitter 感知）
    require("mini.ai").setup({
      n_lines = 500,
    })

    -- 注释（gcc 行注释, gc 可视化注释）
    require("mini.comment").setup({
      mappings = {
        comment = "gc",
        comment_line = "gcc",
      },
    })

    -- 缩进作用域
    require("mini.indentscope").setup({
      draw = { delay = 100, animation = require("mini.indentscope").gen_animation.none() },
    })

    -- 图标提供者（其他插件依赖）
    require("mini.icons").setup()

    -- 状态栏（可选，替代 lualine）
    -- require("mini.statusline").setup()
  end,
}
```

> 来源：[mini.nvim 官方文档](https://nvim-mini.org/mini.nvim/)

---

## 3. mini.files — 列式文件管理

替代 neo-tree / oil.nvim 的轻量文件浏览器：

```lua
{
  "nvim-mini/mini.nvim",
  keys = {
    {
      "<leader>e",
      function()
        local minifiles = require("mini.files")
        minifiles.open(vim.api.nvim_buf_get_name(0))
        minifiles.reveal_cwd()
      end,
      desc = "文件浏览器",
    },
  },
}
```

特点：
- 列式布局，类似 ranger / yazi
- 支持 `d` 创建目录、`%` 创建文件、`R` 重命名

---

## 4. mini.pick — 轻量模糊选择器

替代 Telescope 的极简选择器：

```lua
local pick = require("mini.pick")
vim.keymap.set("n", "<leader>ff", function()
  pick.builtin.files({}, { source = { show = "hidden" } })
end, { desc = "查找文件" })
vim.keymap.set("n", "<leader>fg", function()
  pick.builtin.grep_live({}, {})
end, { desc = "实时搜索" })
```

---

## 5. 为什么选择 mini.nvim？

| 优势 | 说明 |
|------|------|
| **一致性** | 所有模块 API 风格统一 |
| **轻量** | 每个模块 < 500 行代码 |
| **独立性** | 哪个模块需要装哪个，不互相依赖 |
| **文档质量** | 每个模块有单独的 help 文件和 README |
| **维护活跃** | echasnovski 持续维护 |

---

## 6. 关键参考

| 资源 | URL |
|------|-----|
| mini.nvim 官方 | https://nvim-mini.org/mini.nvim/ |
| mini.pairs | https://github.com/nvim-mini/mini.nvim/blob/main/readmes/mini-pairs.md |
| mini.surround | https://github.com/nvim-mini/mini.nvim/blob/main/readmes/mini-surround.md |
| mini.files | https://github.com/nvim-mini/mini.nvim/blob/main/readmes/mini-files.md |
| mini.pick | https://github.com/nvim-mini/mini.nvim/blob/main/readmes/mini-pick.md |
