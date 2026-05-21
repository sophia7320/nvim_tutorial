# 文件浏览器 — neo-tree / oil.nvim

> **TL;DR**: neo-tree（传统侧边栏文件树）或 oil.nvim（Vim 原生风格文件管理）。2025 年趋势是 oil.nvim（编辑文件系统如同编辑 buffer）。

---

## 1. neo-tree.nvim — 侧边栏文件树

```lua
-- lua/plugins/file-explorer.lua
{
  "nvim-neo-tree/neo-tree.nvim",
  branch = "v3.x",
  dependencies = {
    "nvim-lua/plenary.nvim",
    "nvim-tree/nvim-web-devicons",
    "MunifTanjim/nui.nvim",
  },
  keys = {
    { "<leader>e", "<cmd>Neotree toggle<CR>", desc = "文件浏览器" },
    { "<leader>E", "<cmd>Neotree reveal<CR>", desc = "定位当前文件" },
  },
  opts = {
    filesystem = {
      filtered_items = {
        visible = true,
        hide_dotfiles = false,
        hide_gitignored = true,
      },
      follow_current_file = { enabled = true },
      use_libuv_file_watcher = true,
    },
    window = {
      position = "left",
      width = 30,
      mappings = {
        ["Y"] = function(state)
          local node = state.tree:get_node()
          local path = node:get_id()
          vim.fn.setreg("+", path)
        end,
      },
    },
    default_component_configs = {
      git_status = { symbols = {
        added = "✚",
        deleted = "✖",
        modified = "",
        renamed = "󰁕",
        untracked = "",
      } },
    },
  },
}
```

---

## 2. oil.nvim — 编辑式文件管理

oil.nvim 将文件系统操作变为 vim 原生编辑体验：

```lua
{
  "stevearc/oil.nvim",
  dependencies = { "nvim-tree/nvim-web-devicons" },
  keys = {
    { "-", "<cmd>Oil<CR>", desc = "打开文件管理器" },
  },
  opts = {
    default_file_explorer = true,
    columns = { "icon" },
    view_options = { show_hidden = true },
    float = { max_width = 90, max_height = 40 },
    -- 文件操作后自动刷新
    delete_to_trash = true,
    skip_confirm_for_simple_edits = true,
  },
}
```

**oil.nvim 的文件操作**（如同编辑普通文本）：
- `<CR>` — 打开文件
- `dd` — 删除文件
- `yy` — 复制文件
- `p` — 粘贴文件
- `R` — 重命名（编辑文件名）
- `a` / `i` — 创建文件/目录

**优势**：不需要学习任何新快捷键，完全重用 Vim 编辑本能。

> 来源：[oil.nvim README](https://github.com/stevearc/oil.nvim)

---

## 3. 选型建议

| 方案 | 适合场景 |
|------|----------|
| **neo-tree** | 习惯 VS Code 侧边栏、需要 Git 状态可视化 |
| **oil.nvim** | 偏好 Vim 原生体验、极简主义 |
| **mini.files** | 已使用 mini.nvim 套件，追求极致轻量 |
| **两者都用** | `-` 打开 oil.nvim，`<leader>e` 打开 neo-tree |

---

## 4. 关键参考

| 资源 | URL |
|------|-----|
| neo-tree.nvim | https://github.com/nvim-neo-tree/neo-tree.nvim |
| oil.nvim | https://github.com/stevearc/oil.nvim |
| mini.files | https://github.com/nvim-mini/mini.nvim/blob/main/readmes/mini-files.md |
