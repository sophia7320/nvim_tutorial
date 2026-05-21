# Neovim 补全系统对比与选型

> **TL;DR**: 2025-2026 年社区已大规模迁移至 **blink.cmp**。它比 nvim-cmp 更快、配置更少、功能更全（内置源 + 签名帮助 + 命令行补全）。新项目推荐 blink.cmp，存量项目可继续使用 nvim-cmp。

---

## 1. 社区共识

**LazyVim** 和 **kickstart.nvim** 均已将 blink.cmp 作为默认/推荐补全引擎。

> "blink.cmp 是更广泛的 Neovim 社区已基本迁移到的补全插件。" — [CodingBrush 2026](https://codingbrush.com/configure-neovim-autocompletion-blink-cmp/)

**关键时间线**：
- 2024 Q3：blink.cmp 首次发布（Rust 后端）
- 2024 Q4：LazyVim 添加 blink.cmp extra
- 2025 Q1：kickstart.nvim 讨论迁移（[#1331](https://github.com/nvim-lua/kickstart.nvim/issues/1331)）
- 2025 Q2：社区达成「新项目用 blink，存量可留 cmp」的共识

---

## 2. nvim-cmp vs blink.cmp 核心对比

| 维度 | nvim-cmp | blink.cmp | 推荐 |
|------|----------|-----------|------|
| **安装复杂度** | 5+ 插件（源 + 片段） | 单插件，核心源内置 | blink |
| **首次配置行数** | 50-100 行 | 10-20 行 | blink |
| **性能** | 60ms debounce | 0.5-4ms/按键 | blink |
| **模糊匹配** | fzf 风格，无容错 | Smith-Waterman + **typo 容错** | blink |
| **排序智能度** | proximity + recency | frecency + proximity | blink |
| **纯 Lua** | ✅ | ⚠️ 依赖 Rust 二进制（可选 Lua 降级） | cmp |
| **成熟度** | 高度稳定 | V2 开发中，有 breaking changes | cmp |
| **命令行补全** | 需 cmp-cmdline | **内置** | blink |
| **终端补全** | 不支持 | **内置**（0.11+） | blink |
| **签名帮助** | 需额外插件 | **内置**（实验性） | blink |
| **自动括号** | 需插件/手动 | **内置**（基于语义 token） | blink |
| **Ghost text** | 需手动开启 | 内置 | blink |

> 来源：[blink.cmp 官方文档](https://cmp.saghen.dev/) | [nvim-cmp 源码](https://github.com/hrsh7th/nvim-cmp/)

---

## 3. blink.cmp 架构

```
Trigger → Sources → Fuzzy → Render
   │         │        │        │
   │    ┌────┴────┐   │   ┌────┴────┐
   │    │ LSP     │   │   │  Menu    │
   │    │ Buffer  │   │   │  Docs    │
   │    │ Path    │   │   │  Ghost   │
   │    │ Snippets│   │   │  Sigs    │
   │    │ Omni    │   │   └──────────┘
   │    └─────────┘   │
   │              Rust FFI
   │          Smith-Waterman
   │          + frecency
   │          + proximity
```

**关键创新**：
1. **Rust 模糊引擎**（可选 Lua 降级）：SIMD 优化，约 FZF 的 6 倍速度
2. **无预过滤**：所有候选项都参与模糊匹配（nvim-cmp 先字符串过滤）
3. **Frecency**：综合使用频率（frequency）和最近使用（recency）排序

---

## 4. 推荐配置（blink.cmp）

```lua
-- lua/plugins/completion.lua
return {
  {
    "saghen/blink.cmp",
    version = "1.*", -- 使用稳定版，避免 V2 breaking changes
    dependencies = {
      "rafamadriz/friendly-snippets", -- 预置代码片段
    },
    opts = {
      -- 键位预设：enter 接受，Tab 仅跳转片段
      keymap = {
        preset = "enter",
        ["<C-y>"] = { "select_and_accept", "fallback" },
      },

      -- 外观
      appearance = {
        nerd_font_variant = "mono",
      },

      -- 补全行为
      completion = {
        -- 自动显示文档
        documentation = { auto_show = true, auto_show_delay_ms = 200 },
        -- 自动括号（基于语义 token）
        accept = { auto_brackets = { enabled = true } },
        -- Ghost text
        ghost_text = { enabled = true },
      },

      -- 源配置
      sources = {
        default = { "lsp", "path", "snippets", "buffer" },
      },

      -- 模糊匹配
      fuzzy = {
        implementation = "prefer_rust", -- 最快速
      },
    },
    -- 允许从其他文件通过 opts_extend 追加源
    opts_extend = { "sources.default" },
  },
}
```

**键位预设对比**：

| 预设 | 接受键 | 特点 | 适用人群 |
|------|--------|------|----------|
| `default` | `<C-y>` | 类似 Vim 内置补全 | Vim 传统用户 |
| `super-tab` | `<Tab>` | 类似 VSCode | VSCode 迁移用户 |
| **`enter`** | `<CR>` | Enter 接受，Tab 仅跳转片段 | **推荐** |

> 来源：[blink.cmp 键位文档](https://cmp.saghen.dev/configuration/keymap.html) | [LazyVim Blink Extra](https://lazyvim.github.io/extras/coding/blink.html)

---

## 5. 片段引擎选型

### 推荐：原生 `vim.snippet` + friendly-snippets

Neovim 0.10+ 内置 `vim.snippet`，blink.cmp 默认使用它。配合 `friendly-snippets` 即可开箱即用。

```lua
-- lua/plugins/snippets.lua（可选，blink.cmp 默认已支持）
return {
  "rafamadriz/friendly-snippets",
  -- blink.cmp 自动发现 friendly-snippets，无需额外配置
}
```

### 其他选择

| 引擎 | 特点 | 何时选用 |
|------|------|----------|
| **原生 vim.snippet** | 零配置，Neovim 0.10+ 内置 | **推荐（默认）** |
| **LuaSnip** | 功能最丰富，支持 choiceNode/dynamicNode/autosnippets | 需要高级片段功能 |
| **mini.snippets** | mini.nvim 生态，轻量 | 已使用 mini.nvim 套件 |
| **nvim-snippets** | 基于 vim.snippet，VSCode 格式管理器 | nvim-cmp 用户迁移过渡 |

---

## 6. nvim-cmp 存量配置参考（备选）

如果你已有 nvim-cmp 配置或需要纯 Lua 方案：

```lua
{
  "hrsh7th/nvim-cmp",
  event = "InsertEnter",
  dependencies = {
    "hrsh7th/cmp-nvim-lsp",        -- LSP 源
    "hrsh7th/cmp-buffer",          -- Buffer 源
    "hrsh7th/cmp-path",            -- 路径源
    "hrsh7th/cmp-cmdline",         -- 命令行源
    "L3MON4D3/LuaSnip",            -- 片段引擎
    "saadparwaiz1/cmp_luasnip",    -- 片段源
    "rafamadriz/friendly-snippets",
  },
  config = function()
    local cmp = require("cmp")
    cmp.setup({
      snippet = {
        expand = function(args)
          require("luasnip").lsp_expand(args.body)
        end,
      },
      mapping = cmp.mapping.preset.insert({
        ["<C-b>"] = cmp.mapping.scroll_docs(-4),
        ["<C-f>"] = cmp.mapping.scroll_docs(4),
        ["<C-y>"] = cmp.mapping.confirm({ select = true }),
        ["<C-e>"] = cmp.mapping.abort(),
      }),
      sources = cmp.config.sources({
        { name = "nvim_lsp" },
        { name = "luasnip" },
      }, {
        { name = "buffer" },
        { name = "path" },
      }),
      experimental = { ghost_text = true },
    })
  end,
}
```

---

## 7. 从 nvim-cmp 迁移到 blink.cmp

| nvim-cmp 配置项 | blink.cmp 等价 |
|----------------|----------------|
| `sources = { { name = "nvim_lsp" } }` | `sources.default = { "lsp" }` |
| `sources = { { name = "buffer" } }` | `sources.default = { "buffer" }` |
| `sources = { { name = "luasnip" } }` | `sources.default = { "snippets" }` |
| `cmp.mapping.confirm({ select = true })` | `preset = "enter"` |
| `cmp.mapping.complete()` | `cmp.show()` |
| `cmp.mapping.abort()` | `cmp.hide()` / `cmp.cancel()` |
| `experimental.ghost_text = true` | `completion.ghost_text.enabled = true` |
| `window.documentation = { ... }` | `completion.documentation.auto_show = true` |

**迁移工具**：`blink.compat` 可兼容 nvim-cmp 生态的第三方源。

---

## 8. 关键参考

| 资源 | URL |
|------|-----|
| blink.cmp 官方文档 | https://cmp.saghen.dev/ |
| blink.cmp GitHub | https://github.com/saghen/blink.cmp |
| nvim-cmp GitHub | https://github.com/hrsh7th/nvim-cmp |
| blink.cmp vs nvim-cmp (作者对比) | https://gist.github.com/saghen/e731f6f6e30a4c01f6bc7cdaa389d463 |
| LazyVim Blink Extra | https://lazyvim.github.io/extras/coding/blink.html |
| kickstart 迁移讨论 | https://github.com/nvim-lua/kickstart.nvim/issues/1331 |
| friendly-snippets | https://github.com/rafamadriz/friendly-snippets |
