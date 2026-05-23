# Bufferline + Dashboard + Notify + Indent Guides — UI 扩展

> **TL;DR**: 补充主要 UI 组件：bufferline（缓冲区标签）、dashboard（启动页）、nvim-notify（通知美化）、indent-blankline（缩进线）。

---

## 1. bufferline.nvim — 缓冲区标签栏

```lua
-- lua/plugins/ui.lua
{
  "akinsho/bufferline.nvim",
  event = "VeryLazy",
  dependencies = { "nvim-tree/nvim-web-devicons" },
  keys = {
    { "<leader>bp", "<cmd>BufferLineTogglePin<CR>", desc = "固定缓冲区" },
    { "<leader>bP", "<cmd>BufferLineGroupClose ungrouped<CR>", desc = "关闭其他缓冲区" },
  },
  opts = {
    options = {
      mode = "buffers",          -- buffers | tabs [默认值，可选]
      -- 偏移（为 neo-tree 等侧边栏留空）
      offsets = {
        {
          filetype = "neo-tree",
          text = "File Explorer",
          separator = true,
        },
      },
      -- 诊断显示
      diagnostics = "nvim_lsp",
      diagnostics_indicator = function(_, _, diagnostics_dict)
        local s = " "
        for e, n in pairs(diagnostics_dict) do
          local sym = e == "error" and " "
            or (e == "warning" and " " or " ")
          s = s .. sym .. n
        end
        return s
      end,
      -- 快捷键
      numbers = function(opts)
        return string.format("%s·%s", opts.raise(opts.id), opts.lower(opts.ordinal))
      end,
      close_command = "bdelete! %d",
      right_mouse_command = "bdelete! %d",
      left_mouse_command = "buffer %d",
      middle_mouse_command = nil,
      indicator = { style = "icon", icon = "▎" },  -- [默认值，可选]
      buffer_close_icon = "󰅖",
      modified_icon = "●",
      close_icon = "",
      show_tab_indicators = true,
      separator_style = "thin", -- thin | thick | slope | padded_slope [默认值，可选]
      enforce_regular_tabs = false,   -- [默认值，可选]
      always_show_bufferline = true,  -- [默认值，可选]
      sort_by = "insert_at_end",      -- [默认值，可选]
    },
  },
}
```

**快捷键**：
| 按键 | 操作 |
|------|------|
| `<leader>bp` | 固定/取消固定当前 buffer |
| `<leader>bP` | 关闭未固定的 buffers |
| `<leader>1`~`9` | 跳转到第 N 个 buffer |

> 来源：[bufferline.nvim README](https://github.com/akinsho/bufferline.nvim)

---

## 2. dashboard-nvim — 启动页

```lua
{
  "nvimdev/dashboard-nvim",
  event = "VimEnter",
  dependencies = { "nvim-tree/nvim-web-devicons" },
  opts = {
    theme = "hyper", -- hyper | doom
    config = {
      week_header = { enable = true },
      shortcut = {
        { desc = "󰊳 Update", group = "@property", action = "Lazy update", key = "u" },
        { desc = " Files", group = "Label", action = "Telescope find_files", key = "f" },
        { desc = " New file", group = "Label", action = "ene | startinsert", key = "n" },
        { desc = " Recent", group = "Label", action = "Telescope oldfiles", key = "r" },
        { desc = "󰒲 Find word", group = "Label", action = "Telescope live_grep", key = "g" },
        { desc = " Config", group = "Label", action = "e $MYVIMRC", key = "c" },
        { desc = "󰈆 Quit", group = "Label", action = "qa", key = "q" },
      },
      packages = { enable = true },
      project = { enable = true, limit = 8, icon = "󰉋", action = "Telescope find_files cwd=" },
      mru = { limit = 10, icon = "", cwd_only = false },
      footer = {},
    },
  },
}
```

**备选插件**：`goolord/alpha-nvim`（完全可编程的启动页，benchmark 最快）。

---

## 3. nvim-notify — 通知美化

```lua
{
  "rcarriga/nvim-notify",
  event = "VeryLazy",
  opts = {
    stages = "fade_in_slide_out",   -- [默认值，可选]
    timeout = 3000,
    background_colour = "#000000",  -- [默认值，可选]
  },
  config = function(_, opts)
    local notify = require("notify")
    notify.setup(opts)
    vim.notify = notify -- 替换默认通知
  end,
}
```

---

## 4. indent-blankline.nvim v3 — 缩进引导线

```lua
{
  "lukas-reineke/indent-blankline.nvim",
  main = "ibl",
  event = "BufReadPre",
  opts = {
    indent = { char = "│" },       -- [默认值，可选]
    scope = {
      enabled = true,                 -- [默认值，可选]
      show_start = false,             -- [默认值，可选]
      show_end = false,               -- [默认值，可选]
      highlight = { "Function", "Label" },  -- [默认值，可选]
    },
    exclude = {
      filetypes = { "dashboard", "alpha", "help", "lazy" },
    },
  },
}
```

---

## 5. nvim-web-devicons — 文件图标

大多数 UI 插件依赖此图标库：

```lua
{
  "nvim-tree/nvim-web-devicons",
  lazy = true, -- 作为其他插件的依赖自动加载
}
```

---

## 6. nvim-colorizer — 颜色预览

```lua
{
  "norcalli/nvim-colorizer.lua",
  event = "BufReadPre",
  opts = {
    "css", "html", "javascript", "lua", -- 启用颜色预览的文件类型
  },
}
```

---

## 7. 完整 UI 插件汇总

```lua
-- lua/plugins/ui.lua
return {
  -- 配色方案（最先加载）
  { "catppuccin/nvim", name = "catppuccin", lazy = false, priority = 1000, ... },
  -- 状态栏
  { "nvim-lualine/lualine.nvim", event = "VeryLazy", ... },
  -- 缓冲区标签
  { "akinsho/bufferline.nvim", event = "VeryLazy", ... },
  -- 启动页
  { "nvimdev/dashboard-nvim", event = "VimEnter", ... },
  -- 通知
  { "rcarriga/nvim-notify", event = "VeryLazy", ... },
  -- 缩进线
  { "lukas-reineke/indent-blankline.nvim", main = "ibl", event = "BufReadPre", ... },
  -- 颜色预览
  { "norcalli/nvim-colorizer.lua", event = "BufReadPre", ... },
  -- 图标（作为依赖）
  { "nvim-tree/nvim-web-devicons", lazy = true },
}
```
