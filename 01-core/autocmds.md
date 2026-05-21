# vim.api.nvim_create_autocmd — 自动命令

> **TL;DR**: `nvim_create_autocmd()` 是 Neovim 推荐的自动命令 API。始终使用 `group` 参数分组，并用 `{ clear = true }` 防止重复定义。

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

## 4. 自动命令组 (augroup) 最佳实践

### 为什么需要 augroup？

```lua
-- ❌ 没有 group——re-source 文件时会重复创建！
vim.api.nvim_create_autocmd("BufWritePre", {
  callback = function() print("saving...") end,
})

-- ✅ 有 group + clear = true——re-source 时自动清除旧定义
local group = vim.api.nvim_create_augroup("my_group", { clear = true })
vim.api.nvim_create_autocmd("BufWritePre", {
  group = group,
  callback = function() print("saving...") end,
})
```

**`{ clear = true }` 的作用**：创建组时自动清除该组下所有已有的 autocmd。这意味着即使多次 `:source` 配置文件，也不会重复定义。

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
