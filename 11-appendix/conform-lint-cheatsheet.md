# conform.nvim — Formatter 配置速查

> **TL;DR**: 声明 `formatters_by_ft` → 搞定。其余全有合理默认值。详细教程见 `03-lsp/formatting-linting.md §2`。

## formatters_by_ft

| 文件类型 | 推荐 formatter | 备选 |
|----------|---------------|------|
| `lua` | `"stylua"` | — |
| `python` | `"ruff_format"` | `{ "isort", "black" }` |
| `javascript` | `{ "prettierd", "prettier", stop_after_first = true }` | — |
| `typescript` | 同上 | — |
| `go` | `{ "goimports", "gofmt" }` | — |
| `rust` | `"rustfmt"` | — |
| `c` / `cpp` | `"clang-format"` | — |
| `*` (通配) | `"codespell"` | — |
| `_` (兜底) | `"trim_whitespace"` | — |

## 常用 formatter 名称

| Formatter | 适用语言 | Mason 安装 |
|-----------|----------|-----------|
| `stylua` | Lua | `:MasonInstall stylua` |
| `ruff_format` | Python | `:MasonInstall ruff` |
| `prettierd` | JS/TS/CSS/HTML | `:MasonInstall prettierd` |
| `goimports` | Go | `:MasonInstall goimports` |
| `gofmt` | Go | (Go 自带) |
| `rustfmt` | Rust | `rustup component add rustfmt` |
| `clang-format` | C/C++ | `:MasonInstall clang-format` |
| `shfmt` | Shell | `:MasonInstall shfmt` |
| `codespell` | 通用（拼写） | `:MasonInstall codespell` |
| `trim_whitespace` | 通用 | (内置, 无需安装) |

## 配置选项

| 选项 | 类型 | 默认 | 说明 |
|------|------|------|------|
| `formatters_by_ft` | `table` | `{}` | 🟡 必需 |
| `format_on_save` | `boolean\|table` | — | `{ timeout_ms = 500, lsp_format = "fallback" }` |
| `formatters` | `table` | `{}` | 自定义 formatter 参数 |
| `default_format_opts` | `table` | `{}` | `{ lsp_format = "fallback" }` |

## 常用命令

| 命令/按键 | 说明 |
|----------|------|
| `:ConformInfo` | 查看当前 buffer 的 formatter 状态 |
| `<leader>f` | 手动格式化（需自行配置 keymap） |
| 保存时 | 由 `format_on_save` 控制 |
| `gq` 操作符 | 由 `vim.o.formatexpr` 接管 |

---

# nvim-lint — Linter 配置速查

> **TL;DR**: 声明 `linters_by_ft` + 一个 `BufWritePost` autocmd → 搞定。详细教程见 `03-lsp/formatting-linting.md §4`。

## linters_by_ft

| 文件类型 | 推荐 linter |
|----------|------------|
| `lua` | `"luacheck"` |
| `python` | `{ "ruff", "mypy" }` |
| `javascript` / `typescript` | `"eslint_d"` |
| `markdown` | `"vale"` |
| `c` / `cpp` | `"clangtidy"` |

## 最小配置

```lua
{
  "mfussenegger/nvim-lint",
  event = { "BufReadPre", "BufNewFile" },
  config = function()
    local lint = require("lint")
    lint.linters_by_ft = {
      python = { "ruff" },
      lua = { "luacheck" },
    }
    vim.api.nvim_create_autocmd({ "BufWritePost" }, {
      desc = "保存后自动运行 linter",
      callback = function() lint.try_lint() end,
    })
  end,
}
```

## Mason 安装 Linter

| Linter | 语言 | 安装命令 |
|--------|------|----------|
| `luacheck` | Lua | `:MasonInstall luacheck` |
| `ruff` | Python | `:MasonInstall ruff` |
| `mypy` | Python | `:MasonInstall mypy` |
| `eslint_d` | JS/TS | `:MasonInstall eslint_d` |
| `vale` | Markdown | `:MasonInstall vale` |
| `clang-tidy` | C/C++ | (系统包: `apt install clang-tidy`) |
