# 03-lsp — LSP 开发环境配置

## OVERVIEW
Neovim 0.11+ LSP 教程（4 文件）—— Mason 工具安装、LSP 服务器配置、格式化/lint、架构全景。本目录是教程中最重要的章节（README 标记），现已从 1 文件扩展为 4 个深度文件。

## STRUCTURE
```
03-lsp/
├── overview.md              # 架构全景图 + 新旧范式对比 + 推荐阅读入口
├── mason.md                 # mason.nvim 安装器逐句解析（229 行）
├── lspconfig.md             # vim.lsp.config() + vim.lsp.enable() + LspAttach 逐句（390 行）
└── formatting-linting.md    # conform.nvim + nvim-lint 逐句配置（279 行）
```

## WHERE TO LOOK
| Task | File | Section |
|------|------|---------|
| 理解 LSP 整体架构和 0.11 范式变化 | `overview.md` | §1-2 |
| Mason 安装器的每一个配置项 | `mason.md` | §2 (最小配置) |
| mason-tool-installer 自动安装所有工具 | `mason.md` | §7 (最小→高级) |
| vim.lsp.config() 完整服务器配置 | `lspconfig.md` | §2-3 |
| vim.diagnostic.config() 每条选项 | `lspconfig.md` | §3.1 (6 个子项) |
| LspAttach 回调中的所有按键映射 | `lspconfig.md` | §4 (23 个编号注解) |
| conform.nvim 格式化的每条配置 | `formatting-linting.md` | §2 (5 个子块) |
| nvim-lint 代码检查配置 | `formatting-linting.md` | §4 |

## CONVENTIONS (THIS DIR)
- 所有配置代码块使用逐句编号注解 `-- (N) 解释`
- 重复的 lazy.nvim 概念用 `╔══╗` 交叉引用框链接到 02-plugin-manager/
- `overview.md` 不包含逐句注解——它是入口/地图，详细解析在各子文件中
- 每文件末尾有 `## N. 关键参考`

## ANTI-PATTERNS
- 不要使用 `require('lspconfig').xxx.setup()` — 0.11 起改用 `vim.lsp.config()` + `vim.lsp.enable()`
- 不要在多个文件中重复配置 `vim.diagnostic.config()` — 只在 lspconfig.md 中配置一次
