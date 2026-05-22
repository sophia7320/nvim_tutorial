# PROJECT KNOWLEDGE BASE

**Generated:** Thu May 21 2026
**Updated:** Thu May 22 2026 (post-init-deep + 11-appendix cheatsheets)
**Commit:** 6ff5cab
**Branch:** master

## OVERVIEW
Neovim 0.11+ 配置教程文档仓库 — 13 个分类目录、44 个 .md 文件（含 8 个 AGENTS.md 元文件和 36 个内容文件）。纯文档（无代码），面向从零搭建 Neovim IDE 的中文开发者。2026-05 大幅扩展：新增 12-cpp-workflow/ 实战工作流、03-lsp/ 拆分为 4 个深度文件、04-completion/ 新增 blink-cmp.md 逐句解析、02-plugin-manager/ 新增 overview.md 架构入口、全线代码块增加编号逐句注解。

## STRUCTURE
```
./
├── README.md                         # 项目首页（徽章、受众、快速开始）
├── .gitignore                        # 忽略 .sisyphus/、OS 文件
│
├── 00-overview/README.md             # 总览：技术栈、阅读路线、前置条件
├── 01-core/                          # Neovim 原生 API（4 内容文件）
│   ├── init-lua.md                   # init.lua 入口 + 模块加载
│   ├── options.md                    # vim.opt 全局选项（LazyVim 风格，50 条逐句注解）
│   ├── keymaps.md                    # vim.keymap.set + 0.11 默认 LSP 键
│   └── autocmds.md                   # nvim_create_autocmd + LspAttach
├── 02-plugin-manager/                # lazy.nvim（3 内容文件）
│   ├── overview.md                   # 架构全景 + 核心概念 + 文件导航
│   ├── lazy-nvim.md                  # Spec / 懒加载 / 版本控制 / 迁移指南
│   └── directory-structure.md        # 标准目录结构 + 三种权威仓库分析
├── 03-lsp/                           # LSP 开发环境（4 内容文件，最重要）
│   ├── overview.md                   # 架构全景图 + 新旧范式对比
│   ├── mason.md                      # mason.nvim 安装器逐句解析
│   ├── lspconfig.md                  # vim.lsp.config() + LspAttach 逐句解析
│   └── formatting-linting.md         # conform.nvim + nvim-lint 逐句解析
├── 04-completion/                    # 补全系统（2 内容文件）
│   ├── overview.md                   # blink.cmp vs nvim-cmp 深度对比
│   └── blink-cmp.md                  # blink.cmp 完整逐句解析
├── 05-treesitter/                    # Treesitter（1 文件）
├── 06-ui/                            # 配色/状态栏/扩展（3 文件）
├── 07-editing/                       # 编辑增强（5 内容文件，密度最高）
├── 08-dap/                           # nvim-dap 调试（Python/JS/Rust/Go，1 文件）
├── 09-git/                           # gitsigns + Neogit + diffview（1 文件）
├── 10-ai/                            # AI 助手对比（1 文件）
├── 11-appendix/                      # 生态地图、排错、参考资源（7 文件）
│   ├── README.md                    # 生态地图 + 排错速查 + 速查表索引
│   ├── lazy-nvim-spec.md            # lazy.nvim Spec 全字段速查
│   ├── lsp-config.md                # vim.lsp.config/diagnostic.config 速查
│   ├── conform-lint-cheatsheet.md   # conform + nvim-lint 速查
│   ├── blink-cmp-cheatsheet.md      # blink.cmp 全配置键速查
│   ├── mason-cheatsheet.md          # Mason 命令 + 包名速查
│   └── keymap-cheatsheet.md         # 全教程默认键位速查
└── 12-cpp-workflow/                  # C++/CMake/Git 全流程实战（571 行，最大单文件）
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| 找阅读路线/受众定位 | `00-overview/README.md` 或 `README.md` | 内容几乎一致 |
| 修改教程内容 | 对应编号目录下的 .md 文件 | 文件都是独立的（无交叉引用） |
| 添加新章节 | 新建 `NN-topic/` 目录，更新 00-overview 和 README 的目录表 | 编号决定阅读顺序 |
| 查外部参考 URL | 各文件末尾 `## N. 关键参考` 章节 | 19/23 文件有此章节 |
| 找反模式/建议 | `01-core/` 下 3 个文件有结构化「常见错误」章节 | keymaps.md 第 8 节、autocmds.md 第 4 节 |

## CONVENTIONS
- 所有内容文件遵循：`# H1 标题` → `> **TL;DR**: ...` → `---` → `## 1. ...`
- 代码块语言：`lua` 为主（22/30 文件），辅以 `bash`/`vim`/`text`/`json`/`cmake`/`yaml`
- 外部链接使用完整 GitHub permalink 或官方文档 URL，不依赖锚点
- ❌/✅ 标记用于对比旧/新写法（01-core 下有结构化章节，其他文件行内使用）
- 每文件末尾应有 `## N. 关键参考` 参考表格（06-ui/extras.md 例外）
- 编号 00-12 强制执行线性阅读顺序
- **新增（2026-05）**：逐句注解格式——代码块中每条配置右侧 `-- (N) 解释` 编号+中文说明
- **新增（2026-05）**：交叉引用框——重复概念用 `╔══╗` 框链接到首次详解的文件，不重复解析
- 已扩展目录：02-plugin-manager/（3 文件）、03-lsp/（4 文件）、04-completion/（2 文件）、12-cpp-workflow/（全新）、11-appendix/（7 文件，含 6 个速查表）

## ANTI-PATTERNS (THIS PROJECT)
- **禁止使用** `packer.nvim`、`vim-plug`、`lsp-installer`、`coc.nvim` — 全部过时
- **禁止使用** `require('lspconfig').xxx.setup()` — 改用 `vim.lsp.config()` + `vim.lsp.enable()`
- **禁止使用** `vim.cmd("nnoremap ...")` — 改用 `vim.keymap.set()`
- **禁止** 省略 keymap 的 `desc` 属性 — which-key 依赖它
- **禁止** 添加无官方文档/源码引用支撑的技术建议

## COMMANDS
```bash
# 推送更新
git add -A && git commit -m "..." && git push

# 统计文件/行数
find . -name "*.md" -not -path '*/.git/*' | wc -l
find . -name "*.md" -not -path '*/.git/*' -exec wc -l {} + | tail -1

# 检查死链（需安装 lychee）
lychee --base=. --format=markdown .

# 检查 markdown lint（需安装 markdownlint-cli）
markdownlint '**/*.md' --ignore node_modules
```

## NOTES
- LICENSE 文件缺失 — README 徽章链接到不存在的 `LICENSE`
- 无 CI/CD（建议添加 markdown lint + link check）
- 无代码格式化配置（`.editorconfig`、`.markdownlint.yaml`）
- 00-overview/README.md 未遵循标准 TL;DR 格式和编号章节（已知例外但未在约定中显式记录）
- 12-cpp-workflow/README.md 章节编号已统一为 `## 1.`-`## 12.`（原有两个 `## 1.` 标题和无编号标题已修复）
- 仓库根无真正的 `.lua`/`.json` 配置文件 — 所有代码嵌入在 markdown 代码块中
