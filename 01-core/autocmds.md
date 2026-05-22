# vim.api.nvim_create_autocmd — 自动命令

> **TL;DR**: `nvim_create_autocmd()` 是 Neovim 推荐的自动命令 API。**augroup 不是可选项**——它是 autocmd 正确性的硬性前提：防止重复注册、便于调试、保护性能。始终使用 `nvim_create_augroup({ clear = true })` + `group` 参数。

---

## 1. API 速查

```lua
vim.api.nvim_create_autocmd({event}, {opts})
```

| 参数 | 说明 |
|------|------|
| `event` | 事件名（String）或事件列表（table） |
| `opts.group` | 自动命令组（推荐使用 `nvim_create_augroup`） |
| `opts.pattern` | 文件名模式匹配，如 `'*.lua'` `{'*.c', '*.h'}` |
| `opts.buffer` | 仅在指定缓冲区生效 |
| `opts.callback` | Lua 函数（**推荐**） |
| `opts.command` | Vim 命令字符串（**不推荐**，用 callback 代替） |
| `opts.desc` | 描述文字（which-key 友好） |
| `opts.once` | 仅执行一次后自动删除 |

> 来源：[Neovim api.txt](https://github.com/neovim/neovim/blob/v0.11.4/runtime/doc/api.txt)

---

## 2. 推荐全局 Autocmds

```lua
-- ~/.config/nvim/lua/config/autocmds.lua
-- ============================================================================
-- 全局自动命令
-- 参考 LazyVim 最佳实践
-- ============================================================================

-- 辅助函数：创建自动命令组
local function augroup(name)
  return vim.api.nvim_create_augroup("my_" .. name, { clear = true })
end

-- ── Yank 高亮 ─────────────────────────────────────────────
vim.api.nvim_create_autocmd("TextYankPost", {
  group = augroup("highlight_yank"),
  desc = "复制时短暂高亮选区",
  callback = function()
    vim.highlight.on_yank { higroup = "IncSearch", timeout = 150 }
  end,
})

-- ── 回到上次编辑位置 ──────────────────────────────────────
vim.api.nvim_create_autocmd("BufReadPost", {
  group = augroup("last_location"),
  desc = "打开文件时跳转到上次编辑位置",
  callback = function(event)
    local exclude = { "gitcommit", "gitrebase" }
    local buf = event.buf
    if vim.tbl_contains(exclude, vim.bo[buf].filetype) then return end
    local mark = vim.api.nvim_buf_get_mark(buf, '"')
    local lcount = vim.api.nvim_buf_line_count(buf)
    if mark[1] > 0 and mark[1] <= lcount then
      pcall(vim.api.nvim_win_set_cursor, 0, mark)
    end
  end,
})

-- ── 文件外部变更检测 ──────────────────────────────────────
vim.api.nvim_create_autocmd({ "FocusGained", "TermClose", "TermLeave" }, {
  group = augroup("checktime"),
  desc = "焦点恢复时检查文件是否在外部被修改",
  callback = function()
    if vim.o.buftype ~= "nofile" then
      vim.cmd("checktime")
    end
  end,
})

-- ── 自动调整分割窗口大小 ──────────────────────────────────
vim.api.nvim_create_autocmd("VimResized", {
  group = augroup("resize_splits"),
  desc = "窗口大小变化时均匀调整分割窗口",
  callback = function()
    local current_tab = vim.fn.tabpagenr()
    vim.cmd("tabdo wincmd =")
    vim.cmd("tabnext " .. current_tab)
  end,
})

-- ── 自动创建目录 ──────────────────────────────────────────
vim.api.nvim_create_autocmd("BufWritePre", {
  group = augroup("auto_create_dir"),
  desc = "保存文件时自动创建不存在的目录",
  callback = function(event)
    if event.match:match("^%w%w+:[\\/][\\/]") then
      return -- 跳过远程文件
    end
    local file = vim.uv.fs_realpath(event.match) or event.match
    local dir = vim.fn.fnamemodify(file, ":p:h")
    if vim.fn.isdirectory(dir) == 0 then
      vim.fn.mkdir(dir, "p")
    end
  end,
})

-- ── 终端设置 ──────────────────────────────────────────────
vim.api.nvim_create_autocmd("TermOpen", {
  group = augroup("terminal"),
  desc = "终端缓冲区设置",
  callback = function()
    vim.opt_local.number = false
    vim.opt_local.relativenumber = false
    vim.opt_local.signcolumn = "no"
  end,
})

-- ── 文件类型特定缩进 ──────────────────────────────────────
vim.api.nvim_create_autocmd("FileType", {
  group = augroup("indent"),
  pattern = { "python", "rust", "go" },
  desc = "特定语言的缩进设置",
  callback = function()
    vim.opt_local.expandtab = true
  end,
})
vim.api.nvim_create_autocmd("FileType", {
  group = augroup("indent_tabs"),
  pattern = { "make", "go" },
  desc = "Makefile/Go 使用 Tab 缩进",
  callback = function()
    vim.opt_local.expandtab = false
    vim.opt_local.tabstop = 4
    vim.opt_local.shiftwidth = 4
  end,
})
```

> 来源：[LazyVim autocmds.lua](https://github.com/folke/lazyvim/blob/main/lua/lazyvim/config/autocmds.lua) | [kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim/blob/master/init.lua)

---

## 3. LspAttach — 动态 LSP 按键映射

> **Neovim 0.11+ 注意**: 已有内置默认 LSP keymaps（`grn`, `gra`, `grr`, `gri`, `grt`, `K`, `gd`）。以下仅添加**额外的**自定义映射。

```lua
vim.api.nvim_create_autocmd("LspAttach", {
  group = vim.api.nvim_create_augroup("my_lsp_attach", { clear = true }),
  callback = function(event)
    local client = vim.lsp.get_client_by_id(event.data.client_id)
    local bufnr = event.buf

    -- 只有在服务器支持时才设置映射
    if client:supports_method("textDocument/declaration", bufnr) then
      vim.keymap.set("n", "gD", vim.lsp.buf.declaration,
        { buffer = bufnr, desc = "LSP: 跳转到声明" })
    end

    -- 诊断快捷方式
    vim.keymap.set("n", "gl", vim.diagnostic.open_float,
      { buffer = bufnr, desc = "LSP: 显示行诊断" })
    vim.keymap.set("n", "[d", function()
      vim.diagnostic.jump({ count = -1, float = true })
    end, { buffer = bufnr, desc = "LSP: 上一个诊断" })
    vim.keymap.set("n", "]d", function()
      vim.diagnostic.jump({ count = 1, float = true })
    end, { buffer = bufnr, desc = "LSP: 下一个诊断" })

    -- Inlay Hints 切换
    if client:supports_method("textDocument/inlayHint", bufnr) then
      vim.keymap.set("n", "<leader>th", function()
        vim.lsp.inlay_hint.enable(
          not vim.lsp.inlay_hint.is_enabled({ bufnr = bufnr }),
          { bufnr = bufnr }
        )
      end, { buffer = bufnr, desc = "LSP: 切换 Inlay Hints" })
    end
  end,
})
```

---

## 4. 自动命令组 (augroup) — 深入理解

### 4.1 什么是 augroup？

augroup 是 Neovim 中 autocmd 的**组织单元**——你可以把它想象成 autocmd 的「命名空间」。每个 autocmd 要么属于一个命名的 group，要么属于全局默认的「无名组」。

```text
全局 autocmd 注册表
├── (无名组)              ← 没有 group 参数的 autocmd 都在这里
│   ├── BufWritePre *    (你的某个回调)
│   ├── BufWritePre *    (插件的回调)
│   └── ...
├── my_highlight_yank    ← group = "my_highlight_yank"
│   └── TextYankPost *
├── my_last_location     ← group = "my_last_location"
│   └── BufReadPost *
└── ...
```

**augroup 的核心价值**：让你能**以组为单位**管理 autocmd——批量清除、按名称查找、防止冲突。

> 来源：[Neovim autocmd 文档](https://neovim.io/doc/user/autocmd.html#autocmd-groups)

---

### 4.2 如果用 augroup 会怎样？（对比演示）

下面的三个例子帮助你直观感受 augroup 的价值——从「完全不用」到「正确使用」的三级跳。

#### ❌ 第一级：完全不用 group

```lua
-- lua/config/autocmds.lua
vim.api.nvim_create_autocmd("BufWritePre", {
  callback = function()
    print("文件即将保存: " .. vim.fn.expand("%"))
  end,
})
```

**如果你只 source 一次这个文件，它是正常的。** 但 Neovim 配置开发中，你会频繁 `:luafile %` 或 `:source %` 来热重载代码来测试。每次 source，都会在全局无名组里**追加一个新的 autocmd**——旧的不会被清除！

source 3 次后，全局注册表中的状态：

```text
(无名组)
├── BufWritePre *  print("文件即将保存...")   ← 第 1 次 source
├── BufWritePre *  print("文件即将保存...")   ← 第 2 次 source（重复！）
└── BufWritePre *  print("文件即将保存...")   ← 第 3 次 source（重复！）
```

**后果**：每次保存文件，`print()` 被调用 **3 次**。对于轻量回调（如 `print`）只是啰嗦；对于重量回调（如格式化、LSP 操作），会**指数级消耗 CPU**，且你可能完全不知道问题出在哪——因为没有报错，只是越来越慢。

```vim
" 诊断命令：查看某个事件的 autocmd 注册了几次
:autocmd BufWritePre
```

---

#### ⚠️ 第二级：用了 group，但没用 `clear = true`

```lua
local group = vim.api.nvim_create_augroup("my_custom", {})  -- 注意：空的 {}，没用 clear

vim.api.nvim_create_autocmd("BufWritePre", {
  group = group,
  callback = function()
    print("文件即将保存: " .. vim.fn.expand("%"))
  end,
})
```

source 3 次后的注册表：

```text
my_custom
├── BufWritePre *  print("文件即将保存...")   ← 第 1 次 source
├── BufWritePre *  print("文件即将保存...")   ← 第 2 次 source（仍然重复！）
└── BufWritePre *  print("文件即将保存...")   ← 第 3 次 source（仍然重复！）
```

**区别**：现在这些重复的 autocmd 都被「装进了 `my_custom` 这个盒子里」，方便你手动清除：

```lua
-- 手动清除某个组下的所有 autocmd
vim.api.nvim_del_augroup_by_name("my_custom")
```

但仍然没有解决「每次 source 都会重复创建」的问题。你需要**每次手动删除再 source**，开发体验很差。

---

#### ✅ 第三级：group + `clear = true`（推荐）

```lua
-- 关键：{ clear = true }
local group = vim.api.nvim_create_augroup("my_custom", { clear = true })

vim.api.nvim_create_autocmd("BufWritePre", {
  group = group,
  callback = function()
    print("文件即将保存: " .. vim.fn.expand("%"))
  end,
})
```

source 无论多少次，注册表始终保持：

```text
my_custom
└── BufWritePre *  print("文件即将保存...")   ← 始终只有一个！
```

**`{ clear = true }` 的工作原理**：每次执行 `nvim_create_augroup("my_custom", { clear = true })` 时，Neovim 会先**删除该组下所有已有的 autocmd**，再创建（或复用）这个组。所以后续 `nvim_create_autocmd` 添加的是唯一的、全新的 autocmd。

> ╔══════════════════════════════════════════╗
> ║ `{ clear = true }` 是 Neovim 社区的      ║
> ║ **硬共识**。LazyVim、kickstart.nvim、   ║
> ║ NvChad 全部使用此模式。                  ║
> ╚══════════════════════════════════════════╝

---

### 4.3 实战：本教程中的 augroup 辅助函数

所有教程中的 autocmd 配置都遵循同一套约定——用一个辅助函数简化 `nvim_create_augroup` 调用：

```lua
-- 定义在 autocmds.lua 顶部
local function augroup(name)
  return vim.api.nvim_create_augroup("my_" .. name, { clear = true })
end

-- 使用示例
vim.api.nvim_create_autocmd("TextYankPost", {
  group = augroup("highlight_yank"),  -- 展开为 my_highlight_yank
  desc = "复制时短暂高亮选区",
  callback = function()
    vim.highlight.on_yank { higroup = "IncSearch", timeout = 150 }
  end,
})
```

**命名约定**：
- 前缀 `my_` 区分你的配置与插件——避免与插件定义的 augroup 冲突
- 后缀描述功能——如 `highlight_yank`、`last_location`、`auto_create_dir`
- 每个功能独占一个 augroup——方便按需禁用（`nvim_del_augroup_by_name("my_highlight_yank")`）

---

### 4.4 augroup 在配置架构中的地位

| 基础设施 | 作用 | 无组织的后果 |
|----------|------|:---|
| `vim.opt` | 统一管理全局选项 | 散落的 `vim.o.xxx` 不可追溯 |
| `vim.keymap.set()` | 统一管理按键映射 | 映射冲突无法定位 |
| **`nvim_create_augroup`** | **统一管理自动命令** | **重复注册 / 性能退化 / 调试困难** |

augroup 与 `vim.opt`、`vim.keymap` 并列，是 Neovim 配置的**三大核心组织机制**。它不是「可选的优化项」——它是 autocmd 正确性的**硬性前提**。

---

### 4.5 快速自查清单

- [ ] 是否每个 `nvim_create_autocmd` 都有 `group` 参数？
- [ ] `nvim_create_augroup` 是否使用了 `{ clear = true }`？
- [ ] 不同功能的 autocmd 是否用了不同的 augroup 名称？
- [ ] augroup 名称是否有统一的前缀（如 `my_`）以避免和插件冲突？
- [ ] 开发时是否能用 `:autocmd <Event>` 快速验证没有重复注册？

---

## 5. 常用事件速查

| 事件 | 触发时机 |
|------|----------|
| `BufReadPre` / `BufReadPost` | 读取文件之前/之后 |
| `BufNewFile` | 创建新文件 |
| `BufWritePre` / `BufWritePost` | 写入之前/之后 |
| `FileType` | 设置文件类型后 |
| `InsertEnter` / `InsertLeave` | 进入/离开插入模式 |
| `TextYankPost` | 文本被复制后 |
| `VimResized` | 窗口大小改变 |
| `FocusGained` / `FocusLost` | 获得/失去焦点 |
| `TermOpen` / `TermClose` | 终端打开/关闭 |
| `LspAttach` / `LspDetach` | LSP 服务器附加/分离 |
| `User VeryLazy` | lazy.nvim + 所有非延迟插件加载完成后 |
| `ColorScheme` | 配色方案改变后 |

---

## 6. 文件类型特定配置

除了 autocmd，还可以使用 **ftplugin** 目录：

```text
~/.config/nvim/
└── after/
    └── ftplugin/
        ├── python.lua   -- 打开 .py 文件时自动执行
        ├── lua.lua      -- 打开 .lua 文件时自动执行
        └── markdown.lua -- 打开 .md 文件时自动执行
```

```lua
-- after/ftplugin/python.lua
-- 打开 Python 文件时自动设置
vim.opt_local.tabstop = 4
vim.opt_local.shiftwidth = 4
vim.opt_local.textwidth = 88  -- Black 格式化宽度
```

---

## 7. 关键参考

| 资源 | URL |
|------|-----|
| Neovim api.txt - autocmd | https://github.com/neovim/neovim/blob/v0.11.4/runtime/doc/api.txt |
| LazyVim autocmds.lua | https://github.com/folke/lazyvim/blob/main/lua/lazyvim/config/autocmds.lua |
| kickstart.nvim LspAttach | https://github.com/nvim-lua/kickstart.nvim/blob/master/init.lua |
