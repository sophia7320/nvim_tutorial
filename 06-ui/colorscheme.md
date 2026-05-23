# 配色方案 — Neovim 主题配置（逐行注解）

> **TL;DR**: 推荐 Catppuccin、TokyoNight、Rose Pine 三大主题。配色方案应设为 `lazy = false` + `priority = 1000`（最先加载）。

---

## 1. 主题速查表

| 主题 | 风格 | 变体 | 特色 |
|------|------|------|------|
| **Catppuccin** | 柔和暖色调 | 4 (latte/frappe/macchiato/mocha) | 插件集成最全 |
| **TokyoNight** | 冷色调赛博风 | 4 (storm/moon/night/day) | folke 作品，LazyVim 默认 |
| **Rose Pine** | 极简 Soho 风 | 3 (main/moon/dawn) | 柔和护眼 |

---

## 2. 逐行注解：Catppuccin（推荐）

```lua
-- lua/plugins/ui.lua

return {
  {
    "catppuccin/nvim",                  -- (1) 插件标识
    name = "catppuccin",                -- (2) 给插件起短名（lazy.nvim 显示用）

    -- (3) lazy = false — 不懒加载。配色方案必须最先加载，其他插件才能继承颜色
    lazy = false,

    -- (4) priority = 1000 — 加载优先级
    --     默认优先级 50。设为 1000 确保在所有其他插件之前加载
    --     否则 lualine/bufferline 等可能在配色之前初始化，出现颜色错乱
    --     ╔══════════════════════════════════════════════╗
--     ║ priority 语义详见：                           ║
--     ║ → 02-plugin-manager/lazy-nvim.md §5          ║
    --     ╚══════════════════════════════════════════════╝
    priority = 1000,

    opts = {
      -- (5) flavour = "mocha" — 选择口味
      --      latte（亮色）/ frappe（中亮）/ macchiato（中暗）/ mocha（暗色）
      flavour = "mocha",          -- [默认值，可选]

      -- (6) transparent_background = false — 背景是否透明
      --      false：使用主题的纯色背景（推荐，对比度好）
      --      true：背景透明，继承终端颜色，配合终端透明度可实现毛玻璃
      transparent_background = false,  -- [默认值，可选]

      -- (7) highlight_overrides — 自定义高亮组颜色
      --     键 = 口味名，值 = 函数，参数 colors 是主题调色板
      highlight_overrides = {
        mocha = function(colors)
          return {
            -- (7a) 将 Comment 高亮改为 overlay0 颜色（更淡的灰色）
            Comment = { fg = colors.overlay0 },
          }
        end,
      },

      -- (8) integrations — 启用/禁用插件颜色适配
      --     启用后，对应插件会自动使用 Catppuccin 的配色
      integrations = {
        blink_cmp = true,               -- (8a) blink.cmp 补全菜单
        gitsigns = true,                -- (8b) Git 符号列
        treesitter = true,              -- (8c) Treesitter 高亮
        which_key = true,               -- (8d) which-key 弹出菜单
        telescope = { enabled = true }, -- (8e) Telescope 查找器
        indent_blankline = { enabled = true }, -- (8f) 缩进线
        native_lsp = {                  -- (8g) LSP 诊断样式
          enabled = true,
          underlines = {
            errors = { "undercurl" },    -- 错误用波浪下划线
            warnings = { "undercurl" },  -- 警告也用波浪下划线
          },
        },
      },
    },

    -- (9) config — 加载插件时执行
    config = function(_, opts)
      -- (9a) 传入 opts 调用主题的 setup
      require("catppuccin").setup(opts)
      -- (9b) 设置 Neovim 的全局配色方案为该主题
      vim.cmd.colorscheme("catppuccin")
    end,
  },
}
```

---

## 3. TokyoNight 配置（简明）

```lua
{
  "folke/tokyonight.nvim",
  lazy = false,
  priority = 1000,
  opts = {
    style = "storm",          -- (10) storm | moon | night | day [默认值，可选]
    transparent = false,      -- (11) 关闭透明背景 [默认值，可选]
    styles = {
      sidebars = "dark",      -- (12) 侧边栏背景比主窗口更深
      floats = "dark",        -- (13) 浮动窗口也用深色背景
    },
    on_highlights = function(hl, colors)
      -- (14) hl 是所有高亮组的表，可直接修改
      hl.Comment = { fg = colors.blue1, italic = true }
    end,
  },
  config = function(_, opts)
    require("tokyonight").setup(opts)
    vim.cmd.colorscheme("tokyonight")
  end,
}
```

---

## 4. Rose Pine 配置（简明）

```lua
{
  "rose-pine/neovim",
  name = "rose-pine",
  lazy = false,
  priority = 1000,
  opts = {
    variant = "moon",        -- (15) main | moon | dawn
    dark_variant = "moon",   -- (16) 当系统为深色模式时使用的变体
    styles = {
      transparency = false,  -- (17) 关闭透明 [默认值，可选]
    },
  },
  config = function(_, opts)
    require("rose-pine").setup(opts)
    vim.cmd.colorscheme("rose-pine")
  end,
}
```

---

## 5. 多主题共存

```lua
-- (18) 通过 enabled 控制哪个主题激活
return {
  { "catppuccin/nvim", name = "catppuccin", enabled = false, lazy = false, priority = 1000 },
  { "folke/tokyonight.nvim", enabled = false, lazy = false, priority = 1000 },
  { "rose-pine/neovim", name = "rose-pine", enabled = true, lazy = false, priority = 1000 },
}

-- (19) 快捷键即时切换主题
vim.keymap.set("n", "<leader>uc", function()
  vim.cmd.colorscheme("catppuccin")
end, { desc = "切换到 Catppuccin" })
```

---

## 6. 为什么 priority = 1000？

```
加载顺序：priority=1000 的插件最先加载
  ↓
配色方案加载（设置所有高亮组的颜色）
  ↓
其他插件加载（lualine/bufferline/telescope 等初始化）
  ↓
其他插件从颜色系统中读取颜色 = 正确的主题颜色 ✓
```

**如果没有 priority**：其他插件可能在配色之前初始化 → 使用默认 Vim 颜色 → 插件颜色错误。

---

## 7. 透明背景三种写法

```lua
-- (20) Catppuccin
opts = { transparent_background = true }

-- (21) TokyoNight
opts = { transparent = true }

-- (22) Rose Pine
opts = { styles = { transparency = true } }
```

---

## 8. 关键参考

| 资源 | URL |
|------|-----|
| Catppuccin | https://github.com/catppuccin/nvim |
| TokyoNight | https://github.com/folke/tokyonight.nvim |
| Rose Pine | https://github.com/rose-pine/neovim |
