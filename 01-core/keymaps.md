# vim.keymap.set — 按键映射

> **TL;DR**: `vim.keymap.set()` 是 Neovim 0.7+ 推荐的按键映射 API。始终使用 `desc` 参数描述映射功能，方便 which-key 展示。

---

## 1. vim.keymap.set() API

**来源**：[Neovim lua.txt](https://github.com/neovim/neovim/blob/v0.11.4/runtime/doc/lua.txt)

```lua
vim.keymap.set({mode}, {lhs}, {rhs}, {opts})
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `mode` | `string \| table` | 模式：`'n'` `'i'` `'v'` `'x'` `'t'` `'c'` 等，或组合 `{'n', 'v'}` |
| `lhs` | `string` | 按键组合，如 `'<leader>ff'` `'<C-h>'` |
| `rhs` | `string \| function` | 要执行的命令或 Lua 函数 |
| `opts` | `table?` | 可选配置：`buffer` `desc` `silent` `remap` `expr` |

---

## 2. 推荐全局 Keymaps

```lua
-- ~/.config/nvim/lua/config/keymaps.lua
-- ============================================================================
-- 全局按键映射
-- 原则：使用 desc 描述每个映射，which-key 会自动展示
-- ============================================================================

-- ── Leader 键定义 ──────────────────────────────────────────
-- 必须在最前面！（已在 init.lua 中设置）
-- vim.g.mapleader = " "
-- vim.g.maplocalleader = "\\"

-- ── 搜索 ─────────────────────────────────────────────────
vim.keymap.set("n", "<Esc>", "<cmd>nohlsearch<CR>",
  { desc = "清除搜索高亮" })

-- ── 窗口导航 ──────────────────────────────────────────────
vim.keymap.set("n", "<C-h>", "<C-w><C-h>",
  { desc = "移动到左侧窗口" })
vim.keymap.set("n", "<C-l>", "<C-w><C-l>",
  { desc = "移动到右侧窗口" })
vim.keymap.set("n", "<C-j>", "<C-w><C-j>",
  { desc = "移动到下方窗口" })
vim.keymap.set("n", "<C-k>", "<C-w><C-k>",
  { desc = "移动到上方窗口" })

-- ── 窗口大小调整 ──────────────────────────────────────────
vim.keymap.set("n", "<C-Up>", "<cmd>resize +2<CR>",
  { desc = "增加窗口高度" })
vim.keymap.set("n", "<C-Down>", "<cmd>resize -2<CR>",
  { desc = "减少窗口高度" })
vim.keymap.set("n", "<C-Left>", "<cmd>vertical resize -2<CR>",
  { desc = "减少窗口宽度" })
vim.keymap.set("n", "<C-Right>", "<cmd>vertical resize +2<CR>",
  { desc = "增加窗口宽度" })

-- ── 标签页 ────────────────────────────────────────────────
vim.keymap.set("n", "<leader><tab>l", "<cmd>tablast<CR>",
  { desc = "最后一个标签" })
vim.keymap.set("n", "<leader><tab>f", "<cmd>tabfirst<CR>",
  { desc = "第一个标签" })
vim.keymap.set("n", "<leader><tab>n", "<cmd>tabnext<CR>",
  { desc = "下一个标签" })
vim.keymap.set("n", "<leader><tab>p", "<cmd>tabprevious<CR>",
  { desc = "上一个标签" })
vim.keymap.set("n", "<leader><tab>c", "<cmd>tabnew<CR>",
  { desc = "新标签" })
vim.keymap.set("n", "<leader><tab>d", "<cmd>tabclose<CR>",
  { desc = "关闭标签" })

-- ── 快速保存 ──────────────────────────────────────────────
vim.keymap.set("n", "<C-s>", "<cmd>w<CR>",
  { desc = "保存文件" })

-- ── 快速退出 ──────────────────────────────────────────────
vim.keymap.set("n", "<leader>qq", "<cmd>q<CR>",
  { desc = "退出" })

-- ── 终端模式 ──────────────────────────────────────────────
vim.keymap.set("t", "<Esc><Esc>", "<C-\\><C-n>",
  { desc = "退出终端模式" })
```

> 来源：基于 [kickstart.nvim keymaps](https://github.com/nvim-lua/kickstart.nvim/blob/master/init.lua) 和 [LazyVim keymaps](https://github.com/folke/lazyvim/blob/main/lua/lazyvim/config/keymaps.lua)

---

## 3. 映射模式速查

| 缩写 | 模式 | 说明 |
|------|------|------|
| `'n'` | Normal | 正常模式 |
| `'i'` | Insert | 插入模式 |
| `'v'` | Visual + Select | 可视模式（含选择） |
| `'x'` | Visual | 仅可视模式 |
| `'t'` | Terminal | 终端模式 |
| `'c'` | Command-line | 命令行模式 |
| `'o'` | Operator-pending | 操作符等待模式 |

---

## 4. 缓冲区本地映射

```lua
-- 仅在当前缓冲区生效（bufnr 由 LspAttach 回调中的 event.buf 提供）
vim.keymap.set("n", "gd", vim.lsp.buf.definition,
  { buffer = bufnr, desc = "跳转到定义" })

-- 在 LspAttach 回调中使用
vim.api.nvim_create_autocmd("LspAttach", {
  callback = function(ev)
    vim.keymap.set("n", "gd", vim.lsp.buf.definition,
      { buffer = ev.buf, desc = "LSP: 跳转到定义" })
  end,
})
```

---

## 5. 高级映射模式

### 5.1 表达式映射 (expr)

```lua
-- 智能 `<Tab>`：补全可见时选择，否则缩进
vim.keymap.set("i", "<Tab>", function()
  if vim.fn.pumvisible() == 1 then
    return "<C-n>"
  end
  return "<Tab>"
end, { expr = true, desc = "智能 Tab" })
```

### 5.2 映射到 Lua 函数

```lua
vim.keymap.set("n", "<leader>ss", function()
  -- 复杂逻辑可以直接写在这里
  local word = vim.fn.expand("<cword>")
  vim.cmd("Telescope grep_string search=" .. word)
end, { desc = "搜索光标下的单词" })
```

### 5.3 递归映射（不推荐）

```lua
-- 默认行为是非递归（noremap），除非显式设置
vim.keymap.set("n", "n", "nzzzv", { desc = "查找下一个并居中" })
```

---

## 6. 按键描述 (desc) 与 which-key 集成

`desc` 字段是 **which-key.nvim** 自动发现和展示的基础：

```lua
-- ✅ 好的描述——which-key 会显示为 "Find Files"
vim.keymap.set("n", "<leader>ff", "<cmd>Telescope find_files<CR>",
  { desc = "查找文件" })

-- ❌ 没有描述——which-key 只显示原始命令
vim.keymap.set("n", "<leader>ff", "<cmd>Telescope find_files<CR>")
```

**分组约定**（which-key 自动按前缀分组）：

| 前缀 | 用途 |
|------|------|
| `<leader>f` | 文件查找 (find) |
| `<leader>g` | Git 操作 |
| `<leader>l` | LSP 相关 |
| `<leader>s` | 搜索 (search) |
| `<leader>c` | 代码 (code) |
| `<leader>d` | 调试 (debug) |
| `<leader>t` | 开关/终端 (toggle/terminal) |
| `<leader>u` | UI 切换 |

---

## 7. Neovim 0.11+ 内置 LSP 默认 Keymaps

Neovim 0.11 起，以下 LSP 映射**自动创建**，无需手动设置：

| Keymap | 功能 |
|--------|------|
| `grn` | 重命名 |
| `gra` | 代码操作 |
| `grr` | 查找引用 |
| `gri` | 查找实现 |
| `grt` | 类型定义 |
| `K` | 悬停文档 |
| `gd` | 跳转到定义（通过 tagfunc） |
| `[d` / `]d` | 上/下一个诊断 |
| `<C-S>` (Insert) | 签名帮助 |

---

## 8. 常见反模式

### ❌ 不要用 vim.cmd 设置映射

```lua
-- ❌ 旧风格，没有 desc
vim.cmd("nnoremap <leader>ff :Telescope find_files<CR>")

-- ✅ 现代风格
vim.keymap.set("n", "<leader>ff", "<cmd>Telescope find_files<CR>",
  { desc = "查找文件" })
```

### ❌ 不要省略 desc

```lua
-- ❌ which-key 无法展示
vim.keymap.set("n", "<leader>ff", "<cmd>Telescope find_files<CR>")
```

## 9. 关键参考

| 资源 | URL |
|------|-----|
| Neovim lua.txt - keymap | https://github.com/neovim/neovim/blob/v0.11.4/runtime/doc/lua.txt |
| LazyVim keymaps.lua | https://github.com/folke/lazyvim/blob/main/lua/lazyvim/config/keymaps.lua |
| kickstart.nvim keymaps | https://github.com/nvim-lua/kickstart.nvim/blob/master/init.lua |
