# lazy.nvim 生态全景 — Neovim 插件管理架构

> **TL;DR**: lazy.nvim 是 2024-2026 年 Neovim 生态的**唯一标准**插件管理器。默认懒加载、Spec 声明式配置、`lazy-lock.json` 版本锁定、`import` 自动发现——一套机制统一了安装、配置、更新、性能优化。本章提供架构全景，详细配置见子文件。

---

## 1. 架构全景图

```
┌─────────────────────────────────────────────────────┐
│                    Neovim 编辑器                      │
├─────────────────────────────────────────────────────┤
│  init.lua → require("config.lazy")                  │
│              │                                       │
│     ┌────────▼────────┐                              │
│     │   lazy.nvim      │  ← 插件管理器（唯一入口）    │
│     │   lazy.setup()   │                              │
│     └────────┬────────┘                              │
│              │                                        │
│   ┌──────────┼──────────┐                             │
│   │          │          │                             │
│   ▼          ▼          ▼                             │
│ Spec      Lazy      Lock                             │
│ (声明)    Load      (锁定)                            │
│           (按需)                                      │
│   │          │          │                             │
│   ▼          ▼          ▼                             │
│ ┌──────────────────────────────┐                     │
│ │      lua/plugins/*.lua       │  ← 插件描述文件      │
│ │  (返回 LazySpec 表)          │     (自动导入)        │
│ │  ├── ui.lua                  │                     │
│ │  ├── editor.lua              │                     │
│ │  ├── lsp.lua                 │                     │
│ │  └── ...                     │                     │
│ └──────────────────────────────┘                     │
│                                                       │
│ ┌──────────────────────────────┐                     │
│ │  ~/.local/share/nvim/lazy/   │  ← 插件安装目录      │
│ │  ├── lazy.nvim/              │                     │
│ │  ├── telescope.nvim/         │                     │
│ │  └── ...                     │                     │
│ └──────────────────────────────┘                     │
└─────────────────────────────────────────────────────┘
```

---

## 2. 核心概念

| 概念 | 说明 | 深入阅读 |
|------|------|----------|
| **Plugin Spec** | 声明式插件描述表——`{ "owner/repo", opts = {} }`。包含来源、加载条件、配置、依赖 | [lazy-nvim.md §3](lazy-nvim.md) |
| **懒加载 (Lazy Load)** | 默认不加载插件，直到触发 `event`/`cmd`/`keys`/`ft` 条件。93 插件仅 ~11ms 启动 | [lazy-nvim.md §5](lazy-nvim.md) |
| **`lazy-lock.json`** | 记录所有插件的精确 commit SHA。跨机器 `:Lazy restore` 恢复相同版本 | [lazy-nvim.md §7](lazy-nvim.md) |
| **`import` 自动发现** | `import = "plugins"` 自动扫描 `lua/plugins/` 下所有 `.lua` 文件 | [lazy-nvim.md §4](lazy-nvim.md) |
| **`opts` 自动 setup** | 设置 `opts` 后 lazy.nvim 自动推断模块名并调用 `require("plugin").setup(opts)` | [lazy-nvim.md §3.3](lazy-nvim.md) |
| **目录结构** | `lua/config/`（核心设置 + LSP 配置）vs `lua/plugins/`（插件声明） | [directory-structure.md](directory-structure.md) |

---

## 3. 与传统方案对比

| 维度 | lazy.nvim | packer.nvim | vim-plug |
|------|-----------|-------------|----------|
| 状态 | **活跃维护** | ⚠️ 已归档 (2023) | 维护模式 |
| 懒加载 | **默认开启** | 需 `opt = true` | 需 `on` 指令 |
| UI | **内置** `:Lazy` | 无 | 无 |
| 版本锁定 | **`lazy-lock.json`** | `snap` | 无 |
| 配置合并 | **`opts` 自动** | 手动 `config` | 手动 |
| 启动速度 | **~11ms** (93 plugins) | ~30ms | ~80ms |

> ╔══════════════════════════════════════════╗
> ║ 完整对比表与迁移指南见：                    ║
> ║ → lazy-nvim.md §1 + §8                   ║
> ╚══════════════════════════════════════════╝

---

## 4. 本目录文件导航

| 文件 | 内容 | 何时阅读 |
|------|------|----------|
| **`lazy-nvim.md`** | 安装引导、Spec 全字段参考、懒加载策略、版本锁定、迁移指南 | **必读**——lazy.nvim 的完整配置手册 |
| **`directory-structure.md`** | 推荐项目目录结构 + LazyVim / kickstart / NvChad 三种仓库分析 | 理解「配置放在哪、为什么这样拆分」 |

---

## 5. 与其他章节的关系

lazy.nvim 是**所有后续章节的基础设施**。从 03-lsp 开始，每个插件都以 `{ "owner/repo", opts = {} }` 的 Spec 格式声明——这正是本目录定义的标准。

```
02-plugin-manager/     ← 你在这里（插件管理基础设施）
    │
    ├──→ 03-lsp/       ← 所有 LSP 插件以 Spec 格式声明
    ├──→ 04-completion/← blink.cmp 的 lazy.nvim Spec
    ├──→ 05-treesitter/← Treesitter 的 Spec + build 钩子
    ├──→ 06-ui/        ← 配色方案的 priority = 1000
    ├──→ 07-editing/   ← 编辑插件的 event/cmd/keys 懒加载
    └──→ ...           ← 所有后续章节
```

---

## 6. 关键参考

| 资源 | URL |
|------|-----|
| lazy.nvim 官方文档 | https://lazy.folke.io/ |
| Plugin Spec 完整参考 | https://lazy.folke.io/spec |
| 延迟加载详解 | https://lazy.folke.io/spec/lazy_loading |
| 目录结构指南 | https://lazy.folke.io/usage/structuring |
| 迁移指南 | https://lazy.folke.io/usage/migration |
| GitHub 仓库 | https://github.com/folke/lazy.nvim |
| LazyVim 配置示例 | https://github.com/LazyVim/LazyVim/tree/main/lua/lazyvim/plugins |
