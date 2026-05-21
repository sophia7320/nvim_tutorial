# Neovim 0.11+ 现代化配置指南 — 总览

> **面向版本**: Neovim 0.11+ / nightly
> **插件管理器**: lazy.nvim
> **配置语言**: Lua（完全摒弃 Vimscript）
> **风格**: 英文代码 + 中文注解

---

## 为什么选择 Neovim（而非 Vim）？

Neovim 自 0.5 版本（2021 年）起将 **Lua 作为一等公民**，迄今已远超 Vim 的扩展能力：

| 特性 | Vim | Neovim |
|------|-----|--------|
| 配置语言 | Vimscript（慢、笨重） | **Lua**（快、现代、模块化） |
| LSP 客户端 | 需第三方（coc.nvim） | **内置原生 LSP**（0.11+ 大幅增强） |
| Treesitter | 不支持 | **内置集成**，语法树级高亮 |
| 异步任务 | 有限 | **完整 async/await** 支持 |
| 插件生态 | 停滞 | **极活跃**（2024-2026 爆发增长） |
| 启动速度 | 慢 | Lua 字节码缓存，**约 30% 更快** |

> 来源：[Neovim 官方文档](https://neovim.io/doc/user/lua-guide.html)

---

## 本文档覆盖的技术栈

```
┌─────────────────────────────────────────────────┐
│                  init.lua (入口)                  │
├─────────────────────────────────────────────────┤
│  01-core/          │  vim.opt, vim.keymap,      │
│  (核心配置)        │  autocmds, Lua modules      │
├─────────────────────────────────────────────────┤
│  02-plugin-manager/│  lazy.nvim 深度解析         │
├─────────────────────────────────────────────────┤
│  03-lsp/           │  Mason → LSP Config        │
│  (语言服务器)      │  Conform (格式化)           │
│                    │  nvim-lint (代码检查)       │
├─────────────────────────────────────────────────┤
│  04-completion/    │  blink.cmp (推荐)           │
│  (补全系统)        │  nvim-cmp (备选)            │
│                    │  片段引擎 (Snippets)         │
├─────────────────────────────────────────────────┤
│  05-treesitter/    │  语法高亮、文本对象、       │
│  (语法解析)        │  缩进、上下文               │
├─────────────────────────────────────────────────┤
│  06-ui/            │  主题、状态栏、标签栏、     │
│  (用户界面)        │  启动页、图标、缩进线       │
├─────────────────────────────────────────────────┤
│  07-editing/       │  Telescope, which-key,     │
│  (编辑增强)        │  flash.nvim, mini.nvim      │
│                    │  文件浏览器                 │
├─────────────────────────────────────────────────┤
│  08-dap/           │  nvim-dap (调试器)          │
│  (调试)            │  nvim-dap-ui (调试界面)     │
├─────────────────────────────────────────────────┤
│  09-git/           │  gitsigns, Neogit,         │
│  (版本控制)        │  diffview                   │
├─────────────────────────────────────────────────┤
│  10-ai/            │  CodeCompanion (推荐)       │
│  (AI 助手)         │  copilot.lua, avante.nvim   │
├─────────────────────────────────────────────────┤
│  11-appendix/      │  生态地图、排错指南、       │
│  (附录)            │  参考资源                   │
├─────────────────────────────────────────────────┤
│  12-cpp-workflow/  │  C++/CMake/Git 全流程     │
│  (实战工作流)      │  LSP → 构建 → 调试 → 运行 │
└─────────────────────────────────────────────────┘
```

---

## 2025-2026 年 Neovim 生态重大变化

### 1. Neovim 0.11 原生 LSP API
- `vim.lsp.config()` + `vim.lsp.enable()` 替代旧的 `require('lspconfig').xxx.setup()`
- `nvim-lspconfig` 角色变为**配置数据集合**，不再是框架
- 内置 LSP 补全 (`vim.lsp.completion`) 和默认快捷键

### 2. blink.cmp 崛起
- LazyVim、kickstart.nvim 等主流发行版已迁移
- Rust 后端 + 内置源（LSP/Buffer/Path/Snippets），替代 nvim-cmp
- 社区共识：新项目推荐 blink.cmp

### 3. lazy.nvim 统一插件管理
- 完全替代 packer.nvim / vim-plug
- 默认懒加载，启动速度极致优化（folke 的 93 插件配置：**~11ms 启动**）

### 4. AI 编码助手爆发
- CodeCompanion.nvim：统一多后端（Claude/OpenAI/Copilot/Ollama）
- avante.nvim：Cursor 风格的 diff-first 工作流
- 推荐组合：copilot.lua (内联补全) + CodeCompanion (聊天)

### 5. mini.nvim 生态成熟
- 40+ 独立 Lua 模块，相互独立使用
- 可替代大量单功能插件（pairs/surround/comment/files/pick）

---

## 推荐阅读顺序

| 阶段 | 章节 | 内容 |
|------|------|------|
| **第一步** | [01-core/](../01-core/) | 理解 init.lua、vim.opt、vim.keymap、autocmds |
| **第二步** | [02-plugin-manager/](../02-plugin-manager/) | 掌握 lazy.nvim 的目录结构和延迟加载 |
| **第三步** | [03-lsp/](../03-lsp/) | 搭建 LSP 开发环境 |
| **第四步** | [04-completion/](../04-completion/) | 配置代码补全 |
| **第五步** | [05-treesitter/](../05-treesitter/) | 启用语法高亮和文本对象 |
| **第六步** | [06-ui/](../06-ui/) | 美化界面 |
| **第七步** | [07-editing/](../07-editing/) | 增强编辑体验 |
| **进阶** | [08-dap/](../08-dap/) | 配置调试器 |
| **进阶** | [09-git/](../09-git/) | Git 集成 |
| **参考** | [11-appendix/](../11-appendix/) | 生态地图、排错、参考资料 |
| **实战** | [12-cpp-workflow/](../12-cpp-workflow/) | C++/CMake/Git 全流程集成 |

---

## 前置要求

- **Neovim >= 0.11**（推荐 nightly build）
- **Git >= 2.19.0**（支持 partial clone）
- **Nerd Font**（显示图标，推荐 JetBrainsMono Nerd Font）
- **ripgrep**（Telescope 实时搜索）
- **Node.js >= 18**（部分 LSP 服务器需要）
- **C 编译器**（gcc/clang，编译 Treesitter parser）
- **fd**（可选，Telescope 文件查找加速）

---

## 兼容性说明

- 本文档所有配置均针对 **Neovim 0.11+** 编写
- 部分特性（如 `vim.lsp.config()`）需要 0.11+
- 如使用 0.10.x，请参考 [01-core/options.md](../01-core/options.md) 中的兼容性注解
- 所有代码示例均基于 **lazy.nvim** 插件管理器
