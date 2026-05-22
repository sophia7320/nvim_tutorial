# Mason — 工具安装命令速查

> **TL;DR**: Mason 命令 + mason-tool-installer 配置速查。详细教程见 `03-lsp/mason.md`。

## Mason 命令

| 命令 | 说明 |
|------|------|
| `:Mason` | 打开管理面板 |
| `:MasonInstall <pkg>` | 安装：`lua-language-server` |
| `:MasonUninstall <pkg>` | 卸载 |
| `:MasonUpdate` | 更新全部 |
| `:MasonLog` | 查看安装日志 |

## mason-tool-installer 命令

| 命令 | 说明 |
|------|------|
| `:MasonToolsInstall` | 批量安装 `ensure_installed` 中缺失的工具 |
| `:MasonToolsUpdate` | 更新 `ensure_installed` 中的所有工具 |
| `:MasonToolsClean` | 卸载不在 `ensure_installed` 中的工具 |

## mason-tool-installer 配置

| 选项 | 类型 | 默认 | 说明 |
|------|------|------|------|
| `ensure_installed` | `string[]` | `{}` | 🟡 必需：自动安装的工具列表 |
| `auto_update` | `boolean` | `false` | 启动时自动更新 |
| `run_on_start` | `boolean` | `true` | 启动时立即检查 |
| `start_delay` | `number` | `3000` | 延迟检查（ms） |
| `debounce_hours` | `number` | `24` | 自动更新间隔（h） |

## 常用工具包名（:Mason 中显示的名称）

| 类别 | 包名 | 用途 |
|------|------|------|
| **LSP** | `lua-language-server` | Lua |
| | `rust-analyzer` | Rust |
| | `pyright` | Python |
| | `gopls` | Go |
| | `clangd` | C/C++ |
| | `typescript-language-server` | JS/TS |
| **Formatter** | `stylua` | Lua |
| | `prettierd` | JS/TS/CSS |
| | `clang-format` | C/C++ |
| | `goimports` | Go |
| | `shfmt` | Shell |
| | `ruff` | Python |
| **Linter** | `luacheck` | Lua |
| | `eslint_d` | JS/TS |
| | `mypy` | Python |
| **DAP** | `debugpy` | Python |
| | `codelldb` | C/C++/Rust |
| | `js-debug-adapter` | JS/TS |
| | `delve` | Go |
