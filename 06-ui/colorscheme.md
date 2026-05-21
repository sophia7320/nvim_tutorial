# 配色方案 — Neovim 主题配置

> **TL;DR**: 推荐 Catppuccin、TokyoNight、Rose Pine 三大主题。配色方案应设为 `lazy = false` + `priority = 1000`（最先加载）。

---

## 1. 2025-2026 年推荐主题

| 主题 | 风格 | 变体数 | 特点 |
|------|------|--------|------|
| **Catppuccin** | 柔和暖色调 | 4 (latte/frappe/macchiato/mocha) | 大量插件集成，社区最活跃 |
| **TokyoNight** | 冷色调赛博风 | 4 (storm/moon/night/day) | folke 作品，LazyVim 默认 |
| **Rose Pine** | 极简 Soho 风 | 3 (main/moon/dawn) | 独特美学，柔和护眼 |
| Kanagawa | 浮世绘风 | 3 | 高对比度暖色系 |
| OneDark | 经典 VS Code 风格 | 1 | 熟悉感，迁移友好 |
| GitHub Theme | GitHub 配色 | 5 | 与 GitHub UI 一致 |

---

## 2. Catppuccin 配置（推荐）

```lua
-- lua/plugins/ui.lua
return {
  {
    "catppuccin/nvim",
    name = "catppuccin",
    lazy = false,          -- 不懒加载
    priority = 1000,       -- 最先加载，确保其他插件继承颜色
    opts = {
      -- 口味选择
      flavour = "mocha",   -- latte | frappe | macchiato | mocha
      -- 透明背景
      transparent_background = false,
      -- 高亮组覆盖
      highlight_overrides = {
        mocha = function(colors)
          return {
            Comment = { fg = colors.overlay0 },
          }
        end,
      },
      -- 插件集成（自动适配这些插件的配色）
      integrations = {
        blink_cmp = true,
        gitsigns = true,
        neotree = true,
        treesitter = true,
        which_key = true,
        telescope = { enabled = true },
        indent_blankline = { enabled = true },
        native_lsp = {
          enabled = true,
          underlines = {
            errors = { "undercurl" },
            warnings = { "undercurl" },
          },
        },
      },
    },
    config = function(_, opts)
      require("catppuccin").setup(opts)
      vim.cmd.colorscheme("catppuccin")
    end,
  },
}
```

> 来源：[Catppuccin nvim README](https://github.com/catppuccin/nvim)

---

## 3. TokyoNight 配置

```lua
{
  "folke/tokyonight.nvim",
  lazy = false,
  priority = 1000,
  opts = {
    style = "storm",       -- storm | moon | night | day
    transparent = false,
    styles = {
      sidebars = "dark",   -- 侧边栏背景风格
      floats = "dark",     -- 浮动窗口背景风格
    },
    on_highlights = function(hl, colors)
      hl.Comment = { fg = colors.blue1, italic = true }
    end,
  },
  config = function(_, opts)
    require("tokyonight").setup(opts)
    vim.cmd.colorscheme("tokyonight")
  end,
}
```

> 来源：[TokyoNight README](https://github.com/folke/tokyonight.nvim)

---

## 4. Rose Pine 配置

```lua
{
  "rose-pine/neovim",
  name = "rose-pine",
  lazy = false,
  priority = 1000,
  opts = {
    variant = "moon",     -- main | moon | dawn
    dark_variant = "moon",
    styles = {
      transparency = false,
    },
  },
  config = function(_, opts)
    require("rose-pine").setup(opts)
    vim.cmd.colorscheme("rose-pine")
  end,
}
```

---

## 5. 多主题切换

```lua
-- 使用哪个主题时，将其他主题设为 enabled = false
return {
  { "catppuccin/nvim", name = "catppuccin", enabled = false, lazy = false, priority = 1000 },
  { "folke/tokyonight.nvim", enabled = false, lazy = false, priority = 1000 },
  { "rose-pine/neovim", name = "rose-pine", enabled = true, lazy = false, priority = 1000 },
}

-- 或创建快捷键切换
vim.keymap.set("n", "<leader>uc", function()
  vim.cmd.colorscheme("catppuccin")
end, { desc = "Catppuccin" })
```

---

## 6. 透明背景配置

所有主流主题都支持透明背景：

```lua
-- Catppuccin
transparent_background = true

-- TokyoNight
transparent = true

-- Rose Pine
styles = { transparency = true }
```

配合终端的透明背景设置，可实现漂亮的毛玻璃效果。

---

## 7. 为什么 priority = 1000？

```lua
priority = 1000  -- 确保配色方案先于其他插件加载
```

没有 `priority` 时，其他插件（如 lualine、bufferline）可能在配色方案之前加载，导致它们使用默认颜色，后续配色方案加载后颜色不统一。

---

## 8. 关键参考

| 资源 | URL |
|------|-----|
| Catppuccin | https://github.com/catppuccin/nvim |
| TokyoNight | https://github.com/folke/tokyonight.nvim |
| Rose Pine | https://github.com/rose-pine/neovim |
| Kanagawa | https://github.com/rebelot/kanagawa.nvim |
| GitHub Theme | https://github.com/projekt0n/github-nvim-theme |
