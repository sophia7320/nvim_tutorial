# 项目审核提示词 — Neovim 0.11+ 配置教程

> **使用方法**：将此文件内容作为新 Sisyphus 会话的**第一条消息**发送。新会话不会有当前会话的记忆，完全基于提示词进行独立审核。

---

## 你的任务

你是 Sisyphus，一个 AI 编码 agent。你的任务是**审核一个 Neovim 0.11+ 配置教程文档仓库的正确性**。

这是你第一次见到这个项目。你对它没有任何预设知识、没有路径依赖、没有"上次的决定"。你的唯一目标是在**代码和技术内容的正确性**上找出问题并修正。

---

## 项目位置

```
/home/sophia/Desktop/nvim_study
```

## 项目概述

这是一个 Neovim 0.11+ 配置学习资料的文档仓库。13 个分类目录，约 30 个 `.md` Markdown 文件。**注意：这是纯文档项目（不是代码项目），所有 Lua 配置以 Markdown 代码块形式嵌入。** 仓库根目录没有任何 `.lua` 文件。

阅读入口是 `README.md`。不要依赖它进行审核——它是面向人类读者的介绍，不是权威。

---

## 第一步：理解项目约定（必读）

**先阅读以下文件**，它们是项目的"宪法"，定义了所有内容文件必须遵守的规则：

1. **`AGENTS.md`** — 项目知识库。注意 `CONVENTIONS` 和 `ANTI-PATTERNS` 部分。
2. **`00-overview/README.md`** — 项目总览和推荐阅读顺序。

### 关键约定速查

| 约定 | 说明 |
|------|------|
| 文件格式 | `# H1 标题` → `> **TL;DR**: ...` → `---` → `## 1. ...` |
| 代码块语言 | 以 `lua` 为主，辅以 `bash`/`vim`/`text`/`json`/`cmake`/`yaml` |
| 章节编号 | `## N. 标题`（N 从 1 开始递增） |
| 关键参考 | 每文件末尾应有 `## N. 关键参考` 表格（已知例外：`00-overview/README.md`、`06-ui/extras.md`） |
| 交叉引用 | 重复概念用 `╔══╗` 框链接到首次详解的文件，不重复解析 |
| 逐句注解 | 配置代码块中每条右侧 `-- (N) 解释`（编号从 1 开始，跨同文件多个代码块连续递增） |
| LSP 风格 | 所有 LSP 配置必须用 `vim.lsp.config()` + `vim.lsp.enable()`，严禁 `require('lspconfig').xxx.setup()` |
| 按键映射 | 只用 `vim.keymap.set()`，严禁 `vim.cmd("nnoremap ...")` |
| 插件管理 | 只用 lazy.nvim 格式 `{ "owner/repo", opts = {} }` |
| 来源要求 | 所有技术建议必须有官方文档或 GitHub 源码引用支撑 |

### 项目禁止的内容

- `packer.nvim`、`vim-plug`、`lsp-installer`、`coc.nvim` — 全部过时
- `require('lspconfig').xxx.setup()` — 改用 `vim.lsp.config()` + `vim.lsp.enable()`
- `vim.cmd("nnoremap ...")` — 改用 `vim.keymap.set()`
- 省略 keymap 的 `desc` 属性
- 无官方文档/源码引用支撑的技术建议

---

## 第二步：子目录 AGENTS.md

以下子目录也有自己的 AGENTS.md，定义了该目录的特有约定：

- `01-core/AGENTS.md` — 核心配置约定
- `03-lsp/AGENTS.md` — LSP 教程约定
- `04-completion/AGENTS.md` — 补全教程约定
- `07-editing/AGENTS.md` — 编辑增强约定
- `12-cpp-workflow/AGENTS.md` — C++ 工作流约定

**审核每个子目录时，先读它的 AGENTS.md，然后逐文件检查其内容是否遵守了该 AGENTS.md 的规定。**

---

## 第三步：正确性审核清单

### A. Lua 语法正确性（高优先级）

对**每一个**包含 `lua` 代码块的文件，检查：
- [ ] Lua 语法是否合法？（没有语法错误、不匹配的括号、未闭合的字符串）
- [ ] `require()` 路径是否与文件结构一致？（`require("config.options")` ↔ `lua/config/options.lua`）
- [ ] Lazy.nvim spec 格式是否正确？（`keys`、`opts`、`dependencies` 等字段的值类型正确）
- [ ] `vim.keymap.set()` 参数是否正确？（模式参数、options 表、desc 字段）
- [ ] `vim.opt` 赋值类型是否正确？（布尔型用 `true/false`，字符串型用引号，数值型用数字）

### B. 插件名和路径正确性（高优先级）

- [ ] 每个 `"owner/repo"` 形式的插件标识符在 GitHub 上是否存在？
- [ ] Mason 服务器名称是否与 [Mason Registry](https://mason-registry.dev/) 一致？
- [ ] conform.nvim 的 `formatters_by_ft` 中使用的 formatter 名称是否正确？（如 `"stylua"` 不是 `"styLua"`）
- [ ] nvim-lint 的 `linters_by_ft` 中使用的 linter 名称是否正确？
- [ ] LSP 服务器名称是否匹配 nvim-lspconfig 中的命名？（如 `"lua_ls"` 不是 `"sumneko_lua"`）
- [ ] blink.cmp 配置键名是否正确？（对照 [官方文档](https://cmp.saghen.dev/)）

### C. Neovim 0.11 API 正确性

- [ ] 所有 `vim.lsp.config()` 调用格式是否正确？（参数：服务器名 + 配置表）
- [ ] 所有 `vim.lsp.enable()` 调用格式是否正确？
- [ ] `vim.diagnostic.config()` 的键名是否正确？（`virtual_text`、`signs`、`underline`、`float`、`severity_sort` 等）
- [ ] `vim.api.nvim_create_autocmd()` 参数是否正确？
- [ ] 默认 LSP 键位映射是否正确？（`grn`、`gra`、`grr`、`gri`、`grt`、`K`、`gd` 等）

### D. 交叉引用正确性

- [ ] 所有 Markdown 相对链接（`[text](../path/file.md)`）指向的文件是否**实际存在**？
- [ ] `╔══╗` 交叉引用框中的章节号引用（如 `§3.2`）是否指向正确的章节？
- [ ] 子目录 AGENTS.md 中的 `WHERE TO LOOK` 表是否与实际文件结构一致？

### E. 逐句注解完整性

- [ ] 新文件（03-lsp/mason.md、03-lsp/lspconfig.md、03-lsp/formatting-linting.md、04-completion/blink-cmp.md）中，每个配置变量是否都有 `-- (N) 解释` 注解？
- [ ] 同文件内编号 `(1)` `(2)` ... 是否**连续无断号**？
- [ ] 注解内容在技术上是否准确？（不误导读者）
- [ ] 重写文件（05-treesitter/treesitter.md、08-dap/dap.md、10-ai/codecompanion.md、01-core/options.md、06-ui/colorscheme.md）的注解是否完整？

### F. 约定一致性

- [ ] 每个内容文件是否以 `# H1 标题` 开头？
- [ ] 每个内容文件是否有 `> **TL;DR**: ...`？
- [ ] 每文件末尾是否有 `## N. 关键参考` 表格？
- [ ] 所有 keymap 是否都有 `desc` 属性？

### G. 内容正确性抽查

随机抽查以下方面的技术准确性（不需要穷举，每个类别查 1-2 处即可）：
- [ ] clangd 命令行参数是否正确？（如 `--background-index`、`--clang-tidy` 确实存在）
- [ ] CMake 命令是否正确？（如 `cmake -B build -DCMAKE_EXPORT_COMPILE_COMMANDS=ON`）
- [ ] nvim-dap 适配器配置是否正确？（`type = "server"`、`port = "${port}"` 格式）
- [ ] blink.cmp 的 source 名称是否正确？（`"lsp"`、`"path"`、`"snippets"`、`"buffer"`）
- [ ] conform.nvim 的 `format_on_save` 配置格式是否正确？

### H. 已知问题复核

复审以下已知问题是否已修复：
- [ ] `01-core/keymaps.md` 第 158-159 行——之前有空 `## 6.` 标题，应已删除
- [ ] `12-cpp-workflow/README.md` 章节编号——之前用 `## 步骤 N：`，应已改为 `## N. `

---

## 第四步：输出要求

### 审核报告格式

```
# Neovim 教程项目审核报告

## 一、发现的问题

### 致命问题（会导致配置无法运行）
| 文件 | 行号 | 问题 | 建议修复 |
|------|------|------|----------|

### 严重问题（技术内容错误）
| 文件 | 行号 | 问题 | 建议修复 |
|------|------|------|----------|

### 约定违反
| 文件 | 问题 | 约定出处 |
|------|------|----------|

### 已修复的问题（二次确认）
| 问题 | 状态 |
|------|------|

## 二、统计
- 审核文件数：N
- 发现问题数：N
- 致命/严重/轻微：N/N/N

## 三、整体评价
（一段话总结项目质量）
```

### 行为准则
- **宁可多报不能漏报** — 如果对一个问题有 1% 的不确定，也要报出来
- **修复致命和严重问题** — 审核完直接修复
- **轻微问题只报告** — 控制修复范围
- **不要重写大幅内容** — 只修复具体错误，不要重构整段
- **质疑一切** — 包括你觉得"显然正确"的配置，去验证它
