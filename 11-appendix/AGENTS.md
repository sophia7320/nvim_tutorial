# 11-appendix — 附录：生态地图、排错、参考资源

## OVERVIEW
附录与速查（7 文件）—— 生态地图、排错指南、插件配置速查表、默认键位速查表。面向已熟悉教程的读者快速查阅，不包含教学性解释。

## STRUCTURE
```
11-appendix/
├── README.md                  # 生态地图 + 排错速查 + 参考资源 + 速查表索引
├── lazy-nvim-spec.md          # lazy.nvim Spec 全字段速查
├── lsp-config.md              # vim.lsp.config() + vim.diagnostic.config() 速查
├── conform-lint-cheatsheet.md # conform.nvim + nvim-lint 配置速查
├── blink-cmp-cheatsheet.md    # blink.cmp 全配置键速查
├── mason-cheatsheet.md        # Mason 命令 + 包名速查
└── keymap-cheatsheet.md       # 全教程默认键位汇总
```

## WHERE TO LOOK
| Task | File | Section |
|------|------|---------|
| 插件生态全景图 | `README.md` | §1 |
| 故障排查速查 | `README.md` | §2 (按问题类型分类) |
| 参考资源汇总 | `README.md` | §4 |
| lazy.nvim Spec 字段查表 | `lazy-nvim-spec.md` | 全文件 |
| LSP 配置选项查表 | `lsp-config.md` | 全文件 |
| formatter/linter 配置查表 | `conform-lint-cheatsheet.md` | 全文件 |
| blink.cmp 配置键查表 | `blink-cmp-cheatsheet.md` | 全文件 |
| Mason 命令/包名查表 | `mason-cheatsheet.md` | 全文件 |
| 默认键位查表 | `keymap-cheatsheet.md` | 全文件 |

## CONVENTIONS (THIS DIR)
- **极简表格驱动**：不含教学性解释，精炼到只给事实
- 面向读者：已通读教程、需要快速查表的熟悉用户
- 所有速查文件遵循 `# H1` → `> **TL;DR**` 格式
- README.md 为索引入口 + 生态地图 + 排错指南
- 速查文件不设「关键参考」——每个字段的值已在教程中验证

## ANTI-PATTERNS
- 不要在速查表中加入教学性解释——这是查表，不是教程
- 不要重复定义已在教程正文中详细解释的概念
- 键位速查表中的来源标记（🔧 内置 / 🛠 自定义）不可省略
