# 04-completion — 代码补全系统

## OVERVIEW
Neovim 补全教程（2 文件）—— blink.cmp（推荐）深度解析 + nvim-cmp vs blink.cmp 对比总览。blink.cmp 是 2025-2026 社区新标准。

## STRUCTURE
```
04-completion/
├── overview.md     # blink.cmp vs nvim-cmp 深度对比 + 社区迁移分析 + 片段引擎选型
└── blink-cmp.md    # blink.cmp 完整逐句解析：keymap/sources/fuzzy/signature 全部参数
```

## WHERE TO LOOK
| Task | File | Section |
|------|------|---------|
| 选择补全系统（blink vs cmp） | `overview.md` | §2 核心对比表 |
| blink.cmp 完整安装配置 | `blink-cmp.md` | §2 (每个参数都有编号注解) |
| blink.cmp 按键预设选哪个 | `blink-cmp.md` | §2 4.1 — preset 四种方案对比 |
| blink.cmp 四个内置源的配置 | `blink-cmp.md` | §2 4.4 — sources 逐项解析 |
| blink.cmp 模糊匹配参数 | `blink-cmp.md` | §2 4.6 — fuzzy 五参数详解 |
| 怎样从 nvim-cmp 迁移 | `overview.md` | §7 迁移对照表 + `blink-cmp.md` §4 |
| 片段引擎怎么选 | `overview.md` | §5 四种引擎对比 |
| LSP capabilities 集成 | `blink-cmp.md` | §3 |

## CONVENTIONS (THIS DIR)
- blink-cmp.md 使用逐句编号注解（与 03-lsp/ 文件风格一致）
- overview.md 不包含详细配置——它是选型指南和对比分析
