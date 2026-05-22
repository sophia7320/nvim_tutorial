# vim.lsp.config() + vim.diagnostic.config() — LSP 配置速查

> **TL;DR**: Neovim 0.11+ LSP 核心 API 速查。详细教程见 `03-lsp/lspconfig.md`。

## vim.lsp.config(name, config)

| key | 类型 | 说明 |
|-----|------|------|
| `cmd` | `string[]` | 启动命令，如 `{ "lua-language-server" }` |
| `filetypes` | `string[]` | 响应的文件类型：`{ "lua" }` |
| `root_markers` | `string[]` | 项目根目录标记：`{ ".git", ".luarc.json" }` |
| `settings` | `table` | 服务器特定 JSON-like 配置 |
| `capabilities` | `table` | LSP 能力集，通常由补全插件提供 |
| `init_options` | `table` | 初始化选项，传给 LSP 服务器 |

**通配符**：`vim.lsp.config("*", { capabilities = ... })` 为所有服务器设置默认值。

## vim.lsp.enable(server)

```lua
vim.lsp.enable("lua_ls")                       -- 单个
vim.lsp.enable({ "lua_ls", "ts_ls", "clangd" }) -- 批量
```

## vim.diagnostic.config()

| key | 类型 | 默认 | 说明 |
|-----|------|------|------|
| `virtual_text` | `boolean\|table` | — | 行尾内联显示；`{ source = "if_many", prefix = "●" }` |
| `virtual_text.spacing` | `number` | `4` | 与代码间距 |
| `signs` | `boolean\|table` | — | 左侧符号列图标 |
| `signs.text` | `table` | — | `{ [severity] = "icon" }` 格式 |
| `underline` | `boolean\|table` | — | 下划线样式 |
| `update_in_insert` | `boolean` | `false` | 插入模式是否刷新 |
| `severity_sort` | `boolean` | `false` | 按严重级别排序 |
| `float.border` | `string` | — | `"rounded"` / `"single"` / `"none"` |
| `float.source` | `boolean` | `false` | 显示来源 |
| `float.max_width` | `number` | — | 浮动窗口最大宽度 |
| `float.max_height` | `number` | — | 浮动窗口最大高度 |

## 常用 LSP 函数

| 函数 | 说明 |
|------|------|
| `vim.lsp.buf.definition()` | 跳转到定义 |
| `vim.lsp.buf.declaration()` | 跳转到声明 |
| `vim.lsp.buf.references()` | 查找引用 |
| `vim.lsp.buf.implementation()` | 查找实现 |
| `vim.lsp.buf.type_definition()` | 类型定义 |
| `vim.lsp.buf.rename()` | 重命名 |
| `vim.lsp.buf.code_action()` | 代码操作 |
| `vim.lsp.buf.hover()` | 悬停文档 |
| `vim.lsp.buf.signature_help()` | 签名帮助 |
| `vim.lsp.buf.document_symbol()` | 文档符号 |
| `vim.lsp.codelens.run()` | 执行 CodeLens |
| `vim.lsp.inlay_hint.enable(bool, { bufnr })` | 切换 inlay hints |
| `vim.lsp.inlay_hint.is_enabled({ bufnr })` | 查询 inlay hints 状态 |

## mason-lspconfig.nvim

| 选项 | 类型 | 默认 | 说明 |
|------|------|------|------|
| `ensure_installed` | `string[]` | `{}` | LSP 服务器名列表 |
| `automatic_enable` | `boolean\|table` | `true` | 自动激活 |
| `handlers` | `table` | `{}` | `["lua_ls"] = function() ... end` |
