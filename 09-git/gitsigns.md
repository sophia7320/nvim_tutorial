# Gitsigns + Neogit + Diffview — Git 集成三件套

> **TL;DR**: 现代 Neovim 的 Git 工作流 = **gitsigns.nvim**（行内标记）+ **Neogit**（Magit 式 UI）+ **diffview.nvim**（差异查看/合并）。

---

## 1. gitsigns.nvim — 行内 Git 标记

```lua
-- lua/plugins/git.lua
{
  "lewis6991/gitsigns.nvim",
  event = { "BufReadPre", "BufNewFile" },
  opts = {
    signs = {
      add = { text = "┃" },
      change = { text = "┃" },
      delete = { text = "_" },       -- [默认值，可选]
      topdelete = { text = "‾" },    -- [默认值，可选]
      changedelete = { text = "~" }, -- [默认值，可选]
      untracked = { text = "┆" },   -- [默认值，可选]
    },
    -- 行内 blame（当前行显示作者和时间）
    current_line_blame = true,
    current_line_blame_opts = {
      virt_text = true,
      virt_text_pos = "eol",
      delay = 1000,
    },
    on_attach = function(bufnr)
      local gs = package.loaded.gitsigns
      -- Hunk 导航
      vim.keymap.set("n", "]h", gs.next_hunk, { buffer = bufnr, desc = "下一个 hunk" })
      vim.keymap.set("n", "[h", gs.prev_hunk, { buffer = bufnr, desc = "上一个 hunk" })
      -- Hunk 操作
      vim.keymap.set("n", "<leader>hs", gs.stage_hunk, { buffer = bufnr, desc = "暂存 hunk" })
      vim.keymap.set("n", "<leader>hr", gs.reset_hunk, { buffer = bufnr, desc = "重置 hunk" })
      vim.keymap.set("v", "<leader>hs", function() gs.stage_hunk({ vim.fn.line("."), vim.fn.line("v") }) end,
        { buffer = bufnr, desc = "暂存选区" })
      vim.keymap.set("n", "<leader>hp", gs.preview_hunk, { buffer = bufnr, desc = "预览 hunk" })
      vim.keymap.set("n", "<leader>hb", gs.blame_line, { buffer = bufnr, desc = "行 Blame" })
    end,
  },
}
```

> 来源：[gitsigns.nvim README](https://github.com/lewis6991/gitsigns.nvim)

---

## 2. Neogit — Magit 风格 Git UI

```lua
{
  "NeogitOrg/neogit",
  dependencies = {
    "nvim-lua/plenary.nvim",
    "sindrets/diffview.nvim", -- 可选的 diff 集成
  },
  keys = {
    { "<leader>gs", "<cmd>Neogit<CR>", desc = "Git 状态" },
  },
  opts = {
    integrations = { diffview = true },
    graph_style = "unicode",
    kind = "tab", -- 在新标签页打开
  },
}
```

**Status Buffer 内快捷键**：
| 按键 | 操作 |
|------|------|
| `s` | Stage（暂存） |
| `S` | Stage unstaged |
| `u` | Unstage |
| `c` | Commit |
| `P` | Push |
| `p` | Pull |
| `b` | Branch |
| `r` | Rebase |

> 来源：[Neogit README](https://github.com/NeogitOrg/neogit)

---

## 3. diffview.nvim — 差异查看与合并

```lua
{
  "sindrets/diffview.nvim",
  cmd = { "DiffviewOpen", "DiffviewFileHistory" },
  keys = {
    { "<leader>gd", "<cmd>DiffviewOpen<CR>", desc = "查看差异" },
    { "<leader>gh", "<cmd>DiffviewFileHistory %<CR>", desc = "文件历史" },
  },
}
```

**合并冲突解决按键**：
| 按键 | 操作 |
|------|------|
| `<leader>co` | 选择 OURS（当前分支） |
| `<leader>ct` | 选择 THEIRS（合并分支） |
| `<leader>cb` | 选择 BASE（共同祖先） |
| `]x` / `[x` | 下一个/上一个冲突 |

> 来源：[diffview.nvim README](https://github.com/sindrets/diffview.nvim)

---

## 4. 推荐快捷键布局

| 前缀 | 用途 |
|------|------|
| `<leader>g` | Git 根菜单 |
| `<leader>gs` | Git 状态 (Neogit) |
| `<leader>gd` | 查看差异 (Diffview) |
| `<leader>gh` | 文件历史 (Diffview) |
| `<leader>gb` | 行 Blame |
| `[h` / `]h` | Hunk 间跳转 |

---

## 5. 系统依赖

```bash
# lazygit（可选，Neogit 的补充 TUI）
sudo apt install lazygit
```

---

## 6. 关键参考

| 资源 | URL |
|------|-----|
| gitsigns.nvim | https://github.com/lewis6991/gitsigns.nvim |
| Neogit | https://github.com/NeogitOrg/neogit |
| diffview.nvim | https://github.com/sindrets/diffview.nvim |
