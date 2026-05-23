# Lualine — 状态栏配置

> **TL;DR**: lualine.nvim 是 Neovim 最流行的状态栏插件，支持主题集成、Git 分支/差异显示、LSP 诊断计数、文件编码等。

---

## 1. 基础配置

```lua
-- lua/plugins/ui.lua
{
  "nvim-lualine/lualine.nvim",
  event = "VeryLazy",
  dependencies = { "nvim-tree/nvim-web-devicons" },
  opts = {
    options = {
      theme = "auto",     -- auto | catppuccin | tokyonight | rose-pine [默认值，可选]
      component_separators = { left = "", right = "" },  -- [默认值，可选]
      section_separators = { left = "", right = "" },    -- [默认值，可选]
      disabled_filetypes = { -- 这些文件类型不显示状态栏
        statusline = { "dashboard", "alpha", "starter" },
      },
      globalstatus = true, -- 全局状态栏（所有窗口共享，推荐）
    },
    sections = {
      -- A 区（左侧第一个）
      lualine_a = { "mode" },
      -- B 区（左侧第二个）
      lualine_b = { "branch", "diff", "diagnostics" },
      -- C 区（左侧中间，通常是文件名）
      lualine_c = {
        {
          "filename",
          path = 1,     -- 1 = 相对路径, 2 = 绝对路径, 0 = 仅文件名
          symbols = {
            modified = "●",  -- 已修改标记
            readonly = "🔒", -- 只读标记
            unnamed = "[No Name]",
          },
        },
      },
      -- X 区（右侧第一个）
      lualine_x = {
        "encoding",
        "fileformat",
        "filetype",
      },
      -- Y 区（右侧第二个）
      lualine_y = { "progress" },
      -- Z 区（右侧最后一个）
      lualine_z = { "location" },
    },

    -- 非活跃窗口的状态栏样式
    inactive_sections = {
      lualine_a = {},
      lualine_b = {},
      lualine_c = { "filename" },
      lualine_x = { "location" },
      lualine_y = {},
      lualine_z = {},
    },
  },
}
```

**分区布局**：
```
┌───────┬─────────┬─────────────────┬──────────┬──────────┬──────────┐
│ mode  │ branch  │   filename      │ encoding │ progress │ location │
│ (A)   │ diff    │   (C)           │  (X)     │   (Y)    │   (Z)    │
│       │ diag (B)│                │          │          │          │
└───────┴─────────┴─────────────────┴──────────┴──────────┴──────────┘
```

> 来源：[lualine.nvim README](https://github.com/nvim-lualine/lualine.nvim)

---

## 2. 自定义组件

```lua
-- 显示 Copilot 状态
lualine_c = {
  {
    function()
      local status = require("copilot.api").status.data
      if status and status.message then
        return "🤖 " .. status.message
      end
      return ""
    end,
    cond = function()
      return package.loaded["copilot"] ~= nil
    end,
  },
  "filename",
},
```

---

## 3. 主题集成

lualine 自动支持主流主题（无需额外配置）：

```lua
-- Catppuccin
options = { theme = "catppuccin" }

-- TokyoNight
options = { theme = "tokyonight" }

-- 自动跟随 vim 配色方案
options = { theme = "auto" }
```

---

## 4. 备选：mini.statusline（极简）

如果你使用 mini.nvim 套件：

```lua
require("mini.statusline").setup({
  content = {
    active = function()
      local mode, mode_hl = MiniStatusline.section_mode({ trunc_width = 120 })
      local git = MiniStatusline.section_git({ trunc_width = 40 })
      local diagnostics = MiniStatusline.section_diagnostics({ trunc_width = 75 })
      local filename = MiniStatusline.section_filename({ trunc_width = 140 })
      local fileinfo = MiniStatusline.section_fileinfo({ trunc_width = 120 })
      local location = MiniStatusline.section_location({ trunc_width = 75 })
      return MiniStatusline.combine_groups({
        { hl = mode_hl, strings = { mode } },
        { hl = "MiniStatuslineDevinfo", strings = { git, diagnostics } },
        "%<",
        { hl = "MiniStatuslineFilename", strings = { filename } },
        "%=",
        { hl = "MiniStatuslineFileinfo", strings = { fileinfo } },
        { hl = mode_hl, strings = { location } },
      })
    end,
  },
})
```

---

## 5. 关键参考

| 资源 | URL |
|------|-----|
| lualine.nvim | https://github.com/nvim-lualine/lualine.nvim |
| mini.statusline | https://github.com/nvim-mini/mini.nvim/blob/main/readmes/mini-statusline.md |
