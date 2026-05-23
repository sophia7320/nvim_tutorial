# vim.opt — Neovim 选项配置（逐行注解）

> **TL;DR**: `vim.opt` 是 Neovim 0.5+ 的 Lua 风格选项 API。本章最核心的 §2 代码块对每一行都做了编号+解释。

---

## 1. vim.opt 基本类型速查

```lua
local opt = vim.opt  -- 别名，减少重复打字

-- 布尔型选项
opt.number = true            -- 显示行号
opt.wrap = false             -- 不自动换行

-- 字符串选项
opt.clipboard = "unnamedplus"         -- 系统剪贴板
opt.completeopt = "menu,menuone,noselect"

-- 数值选项
opt.tabstop = 2              -- Tab 显示为 2 个空格宽度
opt.updatetime = 200         -- CursorHold 触发延迟 (ms)

-- 列表型选项（用 Lua table）
opt.wildignore = { '*.o', '*.a', '__pycache__', '.git' }

-- map 型选项
opt.fillchars = { fold = " ", eob = " " }

-- 追加/移除操作
opt.shortmess:append({ I = true })   -- 不显示启动消息
opt.wildignore:prepend('*.o')
opt.whichwrap:remove({ 'b', 's' })

-- 读取当前值
print(vim.opt.smarttab:get())
```

> 来源：[Neovim lua-guide.txt](https://github.com/neovim/neovim/blob/v0.11.4/runtime/doc/lua-guide.txt)

---

## 2. 推荐默认配置（LazyVim 风格）— 逐行注解

> **每条配置右侧 `—` 后的注释即是逐句解析。** 固定格式：`opt.选项 = 值 -- (编号) 解释`

```lua
-- ~/.config/nvim/lua/config/options.lua
-- 来源：LazyVim options.lua 源码
-- https://github.com/folke/lazyvim/blob/main/lua/lazyvim/config/options.lua

local opt = vim.opt                    -- (1) 别名，减少每次写 vim.opt 的重复

-- ═══════ 编辑行为 ═════════════════════════════════════════
opt.expandtab = true                   -- (2) Tab 键插入空格而非制表符
opt.shiftwidth = 2                     -- (3) >> / << 缩进 2 个空格 [默认值，可选]
opt.tabstop = 2                        -- (4) Tab 字符显示为 2 个空格宽度
opt.smartindent = true                 -- (5) 根据上一行自动判断缩进级别
opt.autoindent = true                  -- (6) 新行继承上一行的缩进

-- ═══════ 行号 ═════════════════════════════════════════════
opt.number = true                      -- (7) 左侧显示绝对行号
opt.relativenumber = true              -- (8) 显示相对行号——跳转时直接看数字按 12j，无需心算偏移

-- ═══════ 视觉 ═════════════════════════════════════════════
opt.termguicolors = true               -- (9)  启用 24-bit 真彩色（所有现代主题的必备前提）
opt.cursorline = true                  -- (10) 高亮光标所在行
opt.cursorlineopt = "number"           -- (11) 仅高亮行号列（而非整行背景），性能更好
opt.signcolumn = "yes"                 -- (12) 始终显示符号列；若设为 "auto"，诊断消失时列宽会跳变
opt.colorcolumn = "80"                 -- (13) 在第 80 列画一条竖线（参考线，许多语言规范要求 ≤80）
opt.scrolloff = 4                      -- (14) 光标距离窗口上/下边缘至少保留 4 行
opt.sidescrolloff = 8                  -- (15) 光标距离窗口左/右边缘至少保留 8 列

-- ═══════ 搜索 ═════════════════════════════════════════════
opt.ignorecase = true                  -- (16) 搜索默认忽略大小写
opt.smartcase = true                   -- (17) 但若搜索词包含大写字母，则自动区分大小写
opt.hlsearch = false                   -- (18) 搜索完成后不高亮所有匹配（按 <Esc> 清除）
opt.incsearch = true                   -- (19) 逐字符匹配时即时跳转 [默认值，可选]
opt.wrapscan = false                   -- (20) 搜索到文件末尾后不回到开头循环

-- ═══════ 窗口 ═════════════════════════════════════════════
opt.splitbelow = true                  -- (21) :split 时新窗口显示在下方
opt.splitright = true                  -- (22) :vsplit 时新窗口显示在右侧
opt.winminwidth = 5                    -- (23) 窗口最小宽度 ≥5 列
opt.fillchars = {                      -- (24) 自定义填充字符（需要 Nerd Font）
  foldopen = "",                      -- (24a) 折叠打开图标
  foldclose = "",                     -- (24b) 折叠关闭图标
  fold = " ",                          -- (24c) 折叠行的填充（空格=不显示填充线）
  foldsep = " ",                       -- (24d) 折叠分隔
  diff = "╱",                          -- (24e) diff 模式下未变更行的填充符号
  eob = " ",                           -- (24f) 文件末尾之后的空白区域（空格=隐藏 ~ 符号）
}

-- ═══════ 文件 ═════════════════════════════════════════════
opt.backup = false                     -- (25) 不创建 *~ 备份文件 [默认值，可选]
opt.writebackup = false                -- (26) 写入时也不创建临时备份
opt.swapfile = false                   -- (27) 禁用 .swp 交换文件（现代推荐——崩溃恢复靠 Git + undofile）
opt.undofile = true                    -- (28) 启用持久化撤销：关闭文件后重开仍可按 u 撤销
opt.undodir = vim.fn.stdpath("data") .. "/undo"  -- (29) 撤销文件的存储目录
opt.mouse = "a"                        -- (30) 允许鼠标点击切换窗口、调整大小、滚动

-- ═══════ 剪贴板 ═══════════════════════════════════════════
opt.clipboard = "unnamedplus"          -- (31) 默认 y/d/p 操作使用系统剪贴板（Linux: + 寄存器）
--      SSH 远程时应禁用，避免延迟：if vim.env.SSH_CONNECTION then opt.clipboard = "" end

-- ═══════ 补全 ═════════════════════════════════════════════
opt.completeopt = "menu,menuone,noselect"
-- (32) menu=弹菜单, menuone=仅一个匹配也弹, noselect=不自动选中第一项
opt.pumblend = 10                      -- (33) 补全弹窗 10% 半透明（0.10+，需要支持透明度的终端）
opt.pumheight = 10                     -- (34) 补全弹窗最多显示 10 行
opt.updatetime = 200                   -- (35) CursorHold 延迟 200ms（默认 4000ms）
--      ╔══════════════════════════════════════════════════╗
--      ║ 降到 200ms 的好处详见 §3.4                         ║
--      ╚══════════════════════════════════════════════════╝

-- ═══════ 折叠 ════════════════════════════════════════════
opt.foldmethod = "expr"                -- (36) 折叠方式：表达式
opt.foldexpr = "v:lua.vim.treesitter.foldexpr()"  -- (37) 使用 Treesitter 语法节点判断折叠位置
opt.foldlevel = 99                     -- (38) 默认展开所有折叠（99=无限深的折叠都展开）
opt.foldenable = false                 -- (39) 打开文件时不自动折叠

-- ═══════ 行末处理 ════════════════════════════════════════
opt.fixendofline = true                -- (40) 保存时自动在文件末尾加空行（POSIX 要求）[默认值，可选]
opt.endofline = true                                           -- [默认值，可选]

-- ═══════ 杂项 ════════════════════════════════════════════
opt.timeoutlen = 300                   -- (41) 按键序列超时 300ms
--      例如：<leader>fd 中的 d 必须在按下 <leader> 后 300ms 内按下，否则视为普通按键
opt.ttimeoutlen = 0                    -- (42) 键码（如 <F5>）超时设为 0=永不等待
opt.conceallevel = 0                   -- (43) 不隐藏 markdown 语法符号 [默认值，可选]
opt.showmode = false                   -- (44) 不显示 "-- INSERT --" 等模式提示
--      因为 lualine/mini.statusline 已在状态栏中显示模式，再显示会冗余
opt.sessionoptions = {                 -- (45) :mksession 保存/恢复的内容
  "buffers", "curdir", "tabpages", "winsize",
  "help", "globals", "skiprtp", "folds",
}
```

---

## 3. 重要选项深入

### 3.1 relativenumber

```lua
opt.relativenumber = true              -- 这个选项的「为什么」见上面 (8)
```

**实战**：你需要移动 12 行 → 看行号 `12` → 按 `12j`。无需心算 `当前行 - 目标行`。

### 3.2 signcolumn = "yes"

```lua
opt.signcolumn = "yes"
```

| 值 | 行为 | 问题 |
|----|------|------|
| `"auto"` | 有诊断时才显示 | **列宽跳变**——文本左右晃动 |
| `"yes"` | 始终显示 | 无跳变，空间稳定 ✅ |

### 3.3 undofile vs swapfile

`swapfile` 是 Vim 6.x 时代的崩溃恢复机制。现代开发环境（Git + 云存储 + 稳定系统）不再需要它。

`undofile`（持久化撤销）让你**关闭文件后重新打开仍可按 `u` 撤销**。

```lua
opt.swapfile = false
opt.undofile = true
opt.undodir = vim.fn.stdpath("data") .. "/undo"
```

### 3.4 updatetime = 200

```lua
opt.updatetime = 200    -- 默认 4000ms（4秒！）对现代编辑器来说太慢
```

降到 200ms 后**同时影响**：
- LSP 诊断出现速度
- gitsigns 行 blame 延迟
- `CursorHold` / `CursorHoldI` 事件触发频率
- Treesitter 上下文更新速度

### 3.5 timeoutlen = 300

```lua
opt.timeoutlen = 300
```

这个值控制 `<leader>` 组合键的容忍时间。设为 300ms 意味着：
- 按下 `<leader>` 后必须在 300ms 内按下一个键，否则视为没按 leader
- 太快可能来不及看 which-key 菜单，太慢影响操作流畅度
- 300ms 是社区共识的甜点值

---

## 4. Neovim 0.11+ 新增选项

| 选项 | 说明 | 推荐值 |
|------|------|--------|
| `pumblend` | 补全弹窗透明度 (0-100) | `10` |
| `smoothscroll` | 平滑滚动（`C-d`/`C-u` 不再跳行） | `true` |
| `splitkeep` | 分割窗口时保持光标位置 | `"screen"` |

```lua
opt.smoothscroll = true     -- (46) 用 C-d/C-u 时逐行平滑滚动
opt.splitkeep = "screen"    -- (47) :split 后光标不跳 [默认值，可选]
```

---

## 5. 条件设置与 vim.g

```lua
-- (48) 仅在 GUI 环境启用——终端内此选项无意义
if vim.fn.has("gui_running") == 1 then
  opt.guifont = "JetBrainsMono Nerd Font:h12"
end

-- (49) SSH 远程时禁用剪贴板——避免每次 yank 都走网络
if vim.env.SSH_CONNECTION then
  opt.clipboard = ""
end

-- (50) vim.g 设置全局变量（非 option 类型的配置）
vim.g.mapleader = " "                  -- Leader 键（必须在所有插件之前设置）
vim.g.loaded_netrw = 1                 -- 禁用内置 netrw（使用 neo-tree/oil.nvim 替代）
vim.g.loaded_netrwPlugin = 1
```

---

## 6. 关键参考

| 资源 | URL |
|------|-----|
| Neovim lua-guide - 选项 | https://neovim.io/doc/user/lua-guide.html#lua-guide-options |
| LazyVim options.lua | https://github.com/folke/lazyvim/blob/main/lua/lazyvim/config/options.lua |
