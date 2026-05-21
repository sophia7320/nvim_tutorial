# 07-editing — 编辑增强插件

## OVERVIEW
编辑体验增强教程（5 文件，全项目密度最高目录）—— 覆盖模糊查找、按键提示、快速跳转、mini.nvim 套件、文件浏览器。这里的插件不涉及 LSP/补全/Treesitter（另有章节），聚焦纯粹的编辑效率提升。

## STRUCTURE
```
07-editing/
├── telescope.md       # Telescope 模糊查找 → pickers 速查 → fzf-lua 备选
├── which-key.md       # which-key v3 → wk.add() API → v2→v3 迁移 → presets
├── flash.md           # flash.nvim → Jump/Treesitter/Remote/Search 四种模式
├── mini-nvim.md       # mini.nvim 套件 → 推荐模块表 → mini.files/pick/pairs 配置
└── file-explorer.md   # neo-tree vs oil.nvim → 选型建议 → 快捷键表
```

## WHERE TO LOOK
| Task | File | Section |
|------|------|---------|
| 配置 Telescope | `telescope.md` | §1 基础配置（含 fzf-native 扩展） |
| 查所有 Telescope pickers | `telescope.md` | §2 核心 Pickers 速查表 |
| which-key v3 新 API | `which-key.md` | §3 v2 → v3 迁移 |
| 四种 flash 跳转模式怎么用 | `flash.md` | §2 核心模式 |
| 哪些 mini 模块替代独立插件 | `mini-nvim.md` | §1 推荐启用模块表 |
| neo-tree 还是 oil.nvim？ | `file-explorer.md` | §3 选型建议 |

## CONVENTIONS (THIS DIR)
- 每个文件末尾有 `## N. 关键参考`（表格格式的 URL 列表）
- 代码块以 `lua` 为主，`bash` 用于系统依赖安装（如 `telescope.md` §5）
- 插件对比用表格（mini.nvim 替代表、neo-tree vs oil.nvim 表）
- TL;DR 行开头，直接给结论

## UNIQUE TO THIS DIR
- `which-key.md` 包含完整的 v2→v3 迁移对比（其他目录无此模式）
- `flash.md` 需注明与 `s` 键的冲突及替代方案
- `mini-nvim.md` 的推荐模块表是项目中最全面的插件对照表
