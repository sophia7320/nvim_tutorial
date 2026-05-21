# Telescope — 模糊查找器

> **TL;DR**: Telescope 是 Neovim 最强大的模糊查找器。核心 pickers：`find_files`、`live_grep`、`buffers`、`help_tags`，以及全套 LSP pickers。依赖 ripgrep 和 fd 获得最佳性能。

---

## 1. 基础配置

```lua
-- lua/plugins/telescope.lua
return {
  {
    "nvim-telescope/telescope.nvim",
    tag = "0.1.8", -- 使用稳定版本
    dependencies = {
      "nvim-lua/plenary.nvim",
      -- 可选的 fzf 原生扩展（大幅提升速度）
      {
        "nvim-telescope/telescope-fzf-native.nvim",
        build = "make",
        cond = function()
          return vim.fn.executable("make") == 1
        end,
      },
    },
    cmd = "Telescope", -- 命令驱动加载
    keys = {
      { "<leader>ff", "<cmd>Telescope find_files<CR>", desc = "查找文件" },
      { "<leader>fg", "<cmd>Telescope live_grep<CR>", desc = "实时搜索" },
      { "<leader>fb", "<cmd>Telescope buffers<CR>", desc = "缓冲区列表" },
      { "<leader>fh", "<cmd>Telescope help_tags<CR>", desc = "帮助文档" },
      { "<leader>fo", "<cmd>Telescope oldfiles<CR>", desc = "最近文件" },
      { "<leader>fk", "<cmd>Telescope keymaps<CR>", desc = "按键映射" },
      { "<leader>fc", "<cmd>Telescope commands<CR>", desc = "命令列表" },
      -- LSP 相关
      { "<leader>fs", "<cmd>Telescope lsp_document_symbols<CR>", desc = "文档符号" },
      { "<leader>fS", "<cmd>Telescope lsp_workspace_symbols<CR>", desc = "工作区符号" },
      { "grr", "<cmd>Telescope lsp_references<CR>", desc = "LSP: 引用" },
      { "<leader>fd", "<cmd>Telescope diagnostics<CR>", desc = "诊断列表" },
    },
    opts = {
      defaults = {
        -- UI 配置
        layout_strategy = "horizontal",
        layout_config = { prompt_position = "top" },
        sorting_strategy = "ascending",
        winblend = 0,
        -- 映射
        mappings = {
          i = {
            ["<C-j>"] = "move_selection_next",
            ["<C-k>"] = "move_selection_previous",
            ["<C-h>"] = "which_key", -- 显示可用快捷键
            ["<C-d>"] = "preview_scrolling_down",
            ["<C-u>"] = "preview_scrolling_up",
          },
        },
      },
      pickers = {
        find_files = {
          hidden = true,       -- 显示隐藏文件
          find_command = {     -- 使用 fd 提升速度
            "fd", "--type", "f", "--strip-cwd-prefix",
          },
        },
        live_grep = {
          additional_args = function()
            return { "--hidden" }
          end,
        },
      },
      extensions = {
        fzf = {
          fuzzy = true,
          override_generic_sorter = true,
          override_file_sorter = true,
          case_mode = "smart_case",
        },
      },
    },
    config = function(_, opts)
      local telescope = require("telescope")
      telescope.setup(opts)
      -- 加载 fzf 扩展
      pcall(telescope.load_extension, "fzf")
    end,
  },
}
```

> 来源：[Telescope README](https://github.com/nvim-telescope/telescope.nvim)

---

## 2. 核心 Pickers 速查

| Picker | 用途 | 快捷键建议 |
|--------|------|------------|
| `find_files` | 按文件名查找 | `<leader>ff` |
| `live_grep` | 全文实时搜索（需 ripgrep） | `<leader>fg` |
| `buffers` | 切换打开的文件 | `<leader>fb` |
| `help_tags` | 搜索帮助文档 | `<leader>fh` |
| `oldfiles` | 最近打开的文件 | `<leader>fo` |
| `keymaps` | 搜索按键映射 | `<leader>fk` |
| `commands` | 搜索 Vim 命令 | `<leader>fc` |
| `lsp_references` | LSP: 查找引用 | `grr` |
| `lsp_definitions` | LSP: 查找定义 | `gd` |
| `lsp_document_symbols` | 当前文件符号 | `<leader>fs` |
| `lsp_workspace_symbols` | 工作区符号 | `<leader>fS` |
| `diagnostics` | 所有诊断列表 | `<leader>fd` |

---

## 3. 常用 UI 主题

```lua
-- 下拉菜单风格
defaults = {
  layout_strategy = "vertical",
  layout_config = { width = 0.9, height = 0.8 },
  theme = "dropdown", -- 或 "ivy", "cursor"
}
```

---

## 4. 备选：fzf-lua

如果你已熟悉终端 fzf，`fzf-lua` 提供更一致的体验：

```lua
{
  "ibhagwan/fzf-lua",
  dependencies = { "nvim-tree/nvim-web-devicons" },
  keys = {
    { "<leader>ff", "<cmd>FzfLua files<CR>", desc = "查找文件" },
    { "<leader>fg", "<cmd>FzfLua live_grep<CR>", desc = "实时搜索" },
  },
}
```

**fzf-lua 优势**：大型代码库中性能更好（利用原生 fzf 二进制）。

---

## 5. 依赖安装

```bash
# 必备
sudo apt install ripgrep    # Telescope live_grep

# 推荐
sudo apt install fd-find    # Telescope find_files 加速
```

---

## 6. 关键参考

| 资源 | URL |
|------|-----|
| Telescope README | https://github.com/nvim-telescope/telescope.nvim |
| Telescope 内置 pickers | https://github.com/nvim-telescope/telescope.nvim/blob/master/doc/telescope.txt |
| fzf-lua | https://github.com/ibhagwan/fzf-lua |
