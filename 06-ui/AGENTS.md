# 06-ui — 用户界面配置

## OVERVIEW
Neovim 界面美化教程（3 文件）—— 配色方案（Catppuccin/TokyoNight/Rose Pine）、状态栏（lualine/mini.statusline）、UI 扩展（bufferline/dashboard/notify/indent-blankline）。本目录覆盖所有视觉相关的插件配置。

## STRUCTURE
```
06-ui/
├── colorscheme.md    # 配色方案（3 大主题逐行注解 + 多主题共存 + 透明背景）
├── statusline.md     # 状态栏（lualine 主力 + mini.statusline 备选）
└── extras.md         # UI 扩展（bufferline / dashboard / notify / indent-blankline / icons）
```

## WHERE TO LOOK
| Task | File | Section |
|------|------|---------|
| Catppuccin 完整逐行配置 | `colorscheme.md` | §2 (9 个子注解) |
| TokyoNight / Rose Pine 简明配置 | `colorscheme.md` | §3-4 |
| 多主题共存和切换 | `colorscheme.md` | §5 |
| lualine 状态栏配置 | `statusline.md` | §1 |
| bufferline 缓冲区标签 | `extras.md` | §1 |
| dashboard-nvim 启动页 | `extras.md` | §2 |
| nvim-notify 通知美化 | `extras.md` | §3 |
| indent-blankline 缩进线 | `extras.md` | §4 |

## CONVENTIONS (THIS DIR)
- colorscheme.md 使用逐句编号注解（与 03-lsp/ 风格一致）
- 配色方案必须设 `lazy = false` + `priority = 1000`（否则颜色错乱）
- 状态栏和其他 UI 插件使用 `event = "VeryLazy"` 延迟加载
- extras.md 不包含 `## N. 关键参考`（根 AGENTS.md 明确列为例外）

## ANTI-PATTERNS
- 配色方案不要懒加载（`lazy = true`）—— 必须在所有其他插件之前加载
- 不要省略配色方案的 `priority = 1000`
- lualine 不要使用硬编码主题名——推荐 `theme = "auto"` 自动跟随