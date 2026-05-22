# 🚀 Neovim 0.11+ 现代化配置完全指南

> **从零搭建你的终极 Neovim IDE —— 基于 2025-2026 年最新官方文档与社区最佳实践**

[![Neovim](https://img.shields.io/badge/Neovim-0.11%2B-%2357A143?logo=neovim&logoColor=white)](https://neovim.io/)
[![Lua](https://img.shields.io/badge/Lua-5.1-%232C2D72?logo=lua&logoColor=white)](https://www.lua.org/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

---

## 📖 关于本教程

这是一份**面向 Neovim 0.11+ / nightly 版本**的现代化配置学习资料，旨在帮助你从零开始，手动搭建一个功能齐全、性能极致、易于维护的 Neovim 开发环境。

### 为什么选择这份教程？

| 对比维度 | 网上多数教程 | 本教程 |
|----------|:-----------:|:------:|
| **信息时效** | 2021-2023 年的过时内容 | **2024-2026 官方文档 + 社区最新共识** |
| **LSP 配置方式** | `require('lspconfig').xxx.setup()` | **`vim.lsp.config()` + `vim.lsp.enable()` (0.11 原生)** |
| **插件管理器** | packer.nvim / vim-plug | **lazy.nvim（唯一现代标准）** |
| **补全系统** | 仅 nvim-cmp | **blink.cmp 深度对比 + 社区迁移趋势分析** |
| **AI 集成** | 过时的 copilot.vim | **CodeCompanion / copilot.lua / avante.nvim 对比** |
| **代码示例** | 零散、不可直接运行 | **每章均提供完整的 `lua/plugins/*.lua` 配置** |
| **目录结构** | 混乱的单文件 | **LazyVim / kickstart / NvChad 三种权威结构分析** |
| **语言** | 纯英文 / 机翻中文 | **英文代码 + 精准中文注解** |

### 核心原则

- **官方文档优先**：所有配置均追溯到 Neovim 官方文档和插件 GitHub README
- **摒弃过时内容**：完全跳过 `lsp-installer`、`packer.nvim`、`coc.nvim`、旧版 LSP 配置模式
- **可运行代码**：每个 `.md` 文件都包含可直接复制使用的配置代码
- **不仅给配置，更讲原理**：每个选择都有"为什么"的解释

---

## 🗂️ 目录结构

```
nvim_study/
│
├── README.md                        ← 你在这里
│
├── 00-overview/                     ← 🌐 总览
│   └── README.md                    ← 技术栈全景、阅读路线、前置要求
│
├── 01-core/                         ← 🔧 核心配置（Neovim 原生）
│   ├── init-lua.md                  ← init.lua 入口、加载顺序、模块机制
│   ├── options.md                   ← vim.opt 全局选项（LazyVim 风格，50 条逐句注解）
│   ├── keymaps.md                   ← vim.keymap.set + 0.11 内置 LSP 默认键
│   └── autocmds.md                  ← nvim_create_autocmd + LspAttach 最佳实践
│
├── 02-plugin-manager/               ← 📦 lazy.nvim
│   ├── overview.md                  ← 架构全景 + 核心概念 + 文件导航
│   ├── lazy-nvim.md                 ← 安装、Spec 全参、懒加载、版本控制、迁移指南
│   └── directory-structure.md       ← 标准目录结构 + 三种权威仓库分析
│
├── 03-lsp/                          ← 🧠 语言服务器（最重要的章节）
│   ├── overview.md                  ← 架构全景 + 新旧范式对比
│   ├── mason.md                     ← mason.nvim 安装器逐句解析
│   ├── lspconfig.md                 ← vim.lsp.config() + vim.lsp.enable() 逐句解析
│   └── formatting-linting.md        ← conform.nvim + nvim-lint 逐句配置
│
├── 04-completion/                   ← ⌨️ 代码补全
│   ├── overview.md                  ← blink.cmp vs nvim-cmp 深度对比 + 选型建议
│   └── blink-cmp.md                 ← blink.cmp 完整逐句解析（keymap/sources/fuzzy）
│
├── 05-treesitter/                   ← 🌳 语法解析
│   └── treesitter.md                ← 安装、高亮、文本对象、上下文
│
├── 06-ui/                           ← 🎨 用户界面
│   ├── colorscheme.md               ← Catppuccin / TokyoNight / Rose Pine 逐句配置
│   ├── statusline.md                ← lualine + mini.statusline
│   └── extras.md                    ← bufferline / dashboard / notify / indent-blankline
│
├── 07-editing/                      ← ⚡ 编辑增强
│   ├── telescope.md                 ← Telescope + fzf-lua 模糊查找
│   ├── which-key.md                 ← which-key v3 按键提示 + v2→v3 迁移
│   ├── flash.md                     ← flash.nvim 四种跳转模式
│   ├── mini-nvim.md                 ← mini.nvim 40+ 模块套件
│   └── file-explorer.md             ← neo-tree vs oil.nvim 文件管理
│
├── 08-dap/                          ← 🐛 调试
│   └── dap.md                       ← nvim-dap + Python/JS/Rust/Go 逐句配置
│
├── 09-git/                          ← 🔀 版本控制
│   └── gitsigns.md                  ← gitsigns + Neogit + diffview
│
├── 10-ai/                           ← 🤖 AI 编码助手
│   └── codecompanion.md             ← copilot.lua + CodeCompanion 逐句解析
│
├── 11-appendix/                     ← 📚 附录
│   └── README.md                    ← 生态地图、故障排查、参考资源汇总
│
└── 12-cpp-workflow/                 ← 🔨 实战工作流
    └── README.md                    ← C++/CMake/Git 全流程：LSP → 构建 → 调试 → 运行
```

---

## 🎯 适合谁？

| 你的情况 | 推荐学习方式 |
|----------|:------------|
| **从零开始**，想搭建自己的配置 | 按编号顺序阅读，边看边写 |
| **已有旧配置**，想迁移到 0.11+ | 重点看 03-lsp（原生 API 迁移）和 02-plugin-manager（从 packer/vim-plug 迁移） |
| **已有配置**，想补充或优化 | 按需翻阅各章节，每章都是独立模块 |
| **使用 LazyVim/NvChad**，想深入理解 | 从 00-overview 开始理解架构，再按需深入 |
| **从 VS Code 迁移** | 先看 00、01、02 理解 Neovim 哲学，再看 03-07 实现 IDE 功能 |

---

## 🚀 快速开始

### 前置要求

```bash
# Neovim >= 0.11（推荐 nightly）
# Ubuntu/Debian
sudo add-apt-repository ppa:neovim-ppa/unstable
sudo apt install neovim

# macOS
brew install neovim --HEAD

# 必备系统工具
sudo apt install git ripgrep fd-find build-essential  # Ubuntu
brew install git ripgrep fd                             # macOS

# Nerd Font（图标显示）
# 推荐: JetBrainsMono Nerd Font
# https://www.nerdfonts.com/font-downloads
```

### 推荐阅读顺序

```
第一步：00-overview  →  理解 Neovim 是什么、本教程覆盖什么
第二步：01-core      →  掌握 init.lua、vim.opt、vim.keymap、autocmds
第三步：02-plugin    →  掌握 lazy.nvim 的目录结构和延迟加载
第四步：03-lsp       →  搭建 LSP 开发环境（最重要的章节！）
第五步：04-completion →  配置代码补全
第六步：05-treesitter →  启用语法高亮和文本对象
第七步：06-ui        →  美化界面
第八步：07-editing   →  增强编辑体验
进阶：08-dap         →  配置调试器
进阶：09-git         →  Git 工作流集成
进阶：10-ai          →  AI 编码助手
参考：11-appendix    →  随时查阅
实战：12-cpp-workflow →  C++ 开发环境全流程集成
```

---

## 📊 覆盖的 2025-2026 关键技术趋势

| 趋势 | 本教程中的体现 |
|------|:-------------|
| **Neovim 0.11 原生 LSP API** | 03-lsp 完整讲解 `vim.lsp.config()` / `vim.lsp.enable()` |
| **blink.cmp 替代 nvim-cmp** | 04-completion 深度对比 + 社区迁移分析 |
| **lazy.nvim 统一插件管理** | 02-plugin-manager 完整参考 |
| **mini.nvim 生态崛起** | 07-editing/mini-nvim.md 40+ 模块速查 |
| **snacks.nvim 整合趋势** | 各章节提及，附录补充 |
| **AI 助手爆发** | 10-ai 四大工具对比 + 推荐组合 |
| **原生 vim.snippet** | 04-completion 片段引擎选型分析 |

---

## 🤝 贡献与反馈

本教程基于大量开源项目和社区讨论。如果你发现：

- 过时或错误的信息
- 缺失的重要主题
- 更好的配置方式

欢迎提 Issue 或 PR！

### 主要参考来源

本教程在撰写过程中参考了以下权威资源：

- [Neovim 官方文档](https://neovim.io/doc/)
- [lazy.nvim 官方文档](https://lazy.folke.io/)
- [blink.cmp 官方文档](https://cmp.saghen.dev/)
- [LazyVim](https://github.com/LazyVim/LazyVim) / [kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim) / [NvChad](https://github.com/NvChad/NvChad)
- [CodeCompanion.nvim](https://codecompanion.olimorris.dev/)
- [mini.nvim](https://nvim-mini.org/mini.nvim/)
- Reddit r/neovim、Neovim Discourse、GitHub Discussions

完整参考列表见 [11-appendix/README.md](11-appendix/README.md)。

---

## 📄 License

MIT
