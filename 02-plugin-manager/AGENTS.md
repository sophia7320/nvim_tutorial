# 02-plugin-manager — lazy.nvim 插件管理器

## OVERVIEW
lazy.nvim 教程（3 文件）—— 架构全景、安装引导、Spec 全参、懒加载策略、版本控制、迁移指南。本目录是整个教程的插件管理基础设施，所有其他章节的插件配置都依赖 lazy.nvim 的 Spec 格式。

## STRUCTURE
```
02-plugin-manager/
├── overview.md            # 架构全景图 + 核心概念 + 与其他章节的关系
├── lazy-nvim.md           # 安装引导 / Spec 全参 / 懒加载 / 版本锁定 / 迁移指南
└── directory-structure.md # 标准目录结构 + LazyVim/kickstart/NvChad 三种权威仓库分析
```

## WHERE TO LOOK
| Task | File | Section |
|------|------|---------|
| 理解 lazy.nvim 整体架构和核心概念 | `overview.md` | §1-2 |
| 本目录文件导航和与其他章节的关系 | `overview.md` | §4-5 |
| lazy.nvim 安装和 bootstrap | `lazy-nvim.md` | §2 安装与引导 |
| Plugin Spec 全部字段 | `lazy-nvim.md` | §3 Plugin Spec 完整参考 |
| 延迟加载策略（event/cmd/keys/ft） | `lazy-nvim.md` | §5 延迟加载策略最佳实践 |
| 版本锁定（lazy-lock.json） | `lazy-nvim.md` | §7 |
| 从 packer/vim-plug 迁移 | `lazy-nvim.md` | §8 迁移指南 |
| 推荐目录结构 | `directory-structure.md` | §1 标准目录结构 |
| 三种权威仓库结构对比 | `directory-structure.md` | §4 真实仓库结构参考 |

## CONVENTIONS (THIS DIR)
- 代码块以 `lua` 为主，`text` 用于目录结构树，`json` 用于 lazy-lock.json
- 插件对比用表格（lazy.nvim vs packer.nvim vs vim-plug §1）
- 迁移指南部分展示旧语法仅用于对比，非推荐用法
- 每文件末尾有 `## N. 关键参考`

## ANTI-PATTERNS
- 不要在 `config` 函数中仅调用 `setup({})` — 用 `opts` 代替
- 不要手动编辑 `lazy-lock.json` — 用 `:Lazy restore` 恢复
- 不要用 `commit = "..."` 固定版本 — 用 `version`/`tag`/`branch` 代替