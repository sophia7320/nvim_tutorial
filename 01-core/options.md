# vim.opt — Neovim 选项配置

> **TL;DR**: `vim.opt` 是 Neovim 0.5+ 提供的 Lua 风格选项设置 API。集中管理所有编辑器行为配置。

---

## 1. vim.opt API 速查

### 1.1 基本语法

```lua
local opt = vim.opt -- 别名，减少重复

-- 布尔型选项
opt.number = true
opt.relativenumber = true
opt.cursorline = true
opt.wrap = false
opt.expandtab = true

-- 字符串选项
opt.clipboard = "unnamedplus"         -- 系统剪贴板
opt.completeopt = "menu,menuone,noselect"
opt.signcolumn = "yes"

-- 数值选项
opt.tabstop = 2
opt.shiftwidth = 2
opt.scrolloff = 4
opt.updatetime = 200
```

> 来源：[Neovim lua-guide.txt](https://github.com/neovim/neovim/blob/v0.11.4/runtime/doc/lua-guide.txt)

### 1.2 列表类型选项

```lua
-- 用 Lua table 设置列表
opt.wildignore = { '*.o', '*.a', '__pycache__', '.git' }

-- map 类型选项
opt.listchars = { space = '_', tab = '>~', trail = '·' }
opt.fillchars = { foldopen = '', foldclose = '', fold = ' ', diff = '╱', eob = ' ' }
```

### 1.3 追加/移除操作

```lua
-- 追加值
opt.shortmess:append({ I = true })      -- 不显示启动消息
opt.wildignore:prepend('*.o')

-- 移除值
opt.whichwrap:remove({ 'b', 's' })

-- 读取当前值
print(vim.opt.smarttab:get())
```

---

## 2. 推荐默认配置（LazyVim 风格）

**来源**：[LazyVim options.lua 源码](https://github.com/folke/lazyvim/blob/main/lua/lazyvim/config/options.lua)

```lua
-- ~/.config/nvim/lua/config/options.lua
-- ============================================================================
-- 全局 Neovim 选项
-- 参考 LazyVim 最佳实践，面向 Neovim 0.11+
-- ============================================================================

local opt = vim.opt

-- ── 编辑行为 ───────────────────────────────────────────────
opt.expandtab = true          -- Tab → 空格
opt.shiftwidth = 2            -- 缩进宽度
opt.tabstop = 2               -- Tab 显示宽度
opt.smartindent = true        -- 智能缩进
opt.autoindent = true         -- 自动缩进

-- ── 行号 ──────────────────────────────────────────────────
opt.number = true             -- 绝对行号
opt.relativenumber = true     -- 相对行号（推荐！跳转时无敌好用）

-- ── 视觉 ──────────────────────────────────────────────────
opt.termguicolors = true      -- 24-bit 真彩色（必须启用）
opt.cursorline = true         -- 高亮当前行
opt.cursorlineopt = "number"  -- 仅行号部分高亮（性能更好）
opt.signcolumn = "yes"        -- 始终显示 signcolumn（防闪烁）
opt.colorcolumn = "80"        -- 80 列参考线
opt.scrolloff = 4             -- 光标上下保留 4 行
opt.sidescrolloff = 8         -- 光标左右保留 8 列

-- ── 搜索 ──────────────────────────────────────────────────
opt.ignorecase = true         -- 搜索忽略大小写
opt.smartcase = true          -- 有大写时自动区分大小写
opt.hlsearch = false          -- 不高亮搜索结果
opt.incsearch = true          -- 增量搜索
opt.wrapscan = false          -- 搜索到底不循环

-- ── 窗口 ──────────────────────────────────────────────────
opt.splitbelow = true         -- 水平分割时新窗口在下方
opt.splitright = true         -- 垂直分割时新窗口在右侧
opt.winminwidth = 5           -- 最小窗口宽度
opt.fillchars = {
  foldopen = "",
  foldclose = "",
  fold = " ",
  foldsep = " ",
  diff = "╱",
  eob = " ",
}

-- ── 文件 ──────────────────────────────────────────────────
opt.backup = false            -- 禁用备份文件
opt.writebackup = false       -- 禁用写入备份
opt.swapfile = false          -- 禁用 swap 文件（现代推荐）
opt.undofile = true           -- 启用持久化撤销（跨会话）
opt.undodir = vim.fn.stdpath("data") .. "/undo"
opt.mouse = "a"               -- 鼠标支持（终端内）

-- ── 剪贴板 ────────────────────────────────────────────────
opt.clipboard = "unnamedplus" -- 系统剪贴板（Linux）

-- ── 补全 ──────────────────────────────────────────────────
opt.completeopt = "menu,menuone,noselect"
opt.pumblend = 10             -- 补全弹窗半透明（0.10+）
opt.pumheight = 10            -- 补全弹窗最大高度
opt.updatetime = 200          -- CursorHold 延迟（影响诊断显示）

-- ── 折叠 ──────────────────────────────────────────────────
opt.foldmethod = "expr"       -- Treesitter 表达式折叠
opt.foldexpr = "v:lua.vim.treesitter.foldexpr()"
opt.foldlevel = 99            -- 默认不折叠
opt.foldenable = false

-- ── 行末处理 ──────────────────────────────────────────────
opt.fixendofline = true       -- 自动在文件末尾加空行
opt.endofline = true

-- ── 杂项 ──────────────────────────────────────────────────
opt.timeoutlen = 300          -- 映射超时（ms）
opt.ttimeoutlen = 0           -- 键码超时
opt.conceallevel = 0          -- 不隐藏 markdown 语法
opt.showmode = false          -- 不显示 -- INSERT --（状态栏已显示）
opt.sessionoptions = { "buffers", "curdir", "tabpages", "winsize", "help", "globals", "skiprtp", "folds" }
```

---

## 3. 重要选项详解

### 3.1 relativenumber（相对行号）

```lua
opt.relativenumber = true
```

**为什么推荐**：当你需要跳转 12 行时，直接看行号 `12` 然后按 `12j`，不需要心算。

### 3.2 signcolumn = "yes"

```lua
opt.signcolumn = "yes"
```

**为什么必须设为 "yes"**：如果设为 "auto"，当没有诊断符号时 signcolumn 会消失，导致文本左右跳动（jitter）。

### 3.3 undofile（持久化撤销）

```lua
opt.undofile = true
```

**为什么比 swapfile 更好**：swapfile 是旧时代的崩溃恢复机制（Vim 6.x），现代文件系统 + Git 已不再需要。`undofile` 让你关闭文件后仍可 `u` 撤销。

### 3.4 剪贴板配置

```lua
-- Linux / macOS
opt.clipboard = "unnamedplus"

-- SSH 远程时禁用（避免延迟）
if vim.env.SSH_CONNECTION then
  opt.clipboard = ""
end
```

### 3.5 updatetime = 200

```lua
opt.updatetime = 200
```

默认 4000ms 太长。降到 200ms 后：
- LSP 诊断更快出现
- `CursorHold` 事件更快触发
- gitsigns blame 更快显示

---

## 4. Neovim 0.11+ 新增选项

| 选项 | 说明 |
|------|------|
| `pumblend` | 补全弹出窗口透明度（0-100） |
| `winblend` | 浮动窗口透明度 |
| `smoothscroll` | 平滑滚动（代替 `C-d`/`C-u` 的跳行） |
| `splitkeep` | 分割窗口时保持光标位置 |

```lua
-- 0.11+ 推荐
opt.smoothscroll = true
opt.splitkeep = "screen"
```

---

## 5. 按条件设置选项

```lua
-- 只在图形界面启用
if vim.fn.has("gui_running") == 1 then
  opt.guifont = "JetBrainsMono Nerd Font:h12"
end

-- 按终端类型
if vim.env.TERM == "xterm-kitty" then
  opt.termguicolors = true
end
```

---

## 6. vim.g 全局变量

某些选项必须通过 `vim.g`（全局变量）设置：

```lua
vim.g.mapleader = " "
vim.g.maplocalleader = "\\"
vim.g.have_nerd_font = true   -- 自定义变量

-- 某些插件使用 vim.g 配置
vim.g.netrw_browse_split = 0
vim.g.loaded_netrw = 1        -- 禁用内置 netrw（使用 neo-tree 替代）
vim.g.loaded_netrwPlugin = 1
```

---

## 7. 关键参考

| 资源 | URL |
|------|-----|
| Neovim lua-guide - 选项 | https://neovim.io/doc/user/lua-guide.html#lua-guide-options |
| LazyVim options.lua | https://github.com/folke/lazyvim/blob/main/lua/lazyvim/config/options.lua |
| kickstart.nvim options | https://github.com/nvim-lua/kickstart.nvim/blob/master/init.lua |
