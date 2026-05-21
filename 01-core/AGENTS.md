# 01-core — Neovim 核心配置

## OVERVIEW
Neovim 原生 API 教程（4 文件）—— 入口、选项、按键、自动命令。本目录是整个教程的基础层，其他所有章节（插件/UI/编辑）都依赖此处的配置。

## STRUCTURE
```
01-core/
├── init-lua.md       # init.lua 入口 → 加载顺序 → 模块机制 → 字节码缓存
├── options.md        # vim.opt 全局选项 → 推荐默认值（LazyVim 风格）→ 条件选项
├── keymaps.md        # vim.keymap.set API → 推荐全局键 → LSP 默认键表 → which-key desc
└── autocmds.md       # nvim_create_autocmd → augroup 最佳实践 → LspAttach → ftplugin
```

## WHERE TO LOOK
| Task | File | Section |
|------|------|---------|
| 理解 init.lua 最小化结构 | `init-lua.md` | §2 最小化模板 |
| 解决加载顺序问题 | `init-lua.md` | §4 加载顺序的重要性 |
| 查 vim.opt 某个选项的写法 | `options.md` | §2 完整默认配置代码块 |
| 查按键映射 API | `keymaps.md` | §1 API 参数表 |
| LSP 键放到哪里 | `autocmds.md` | §3 LspAttach 回调 |
| Augroup 为什么需要 clear=true | `autocmds.md` | §4 自动命令组最佳实践 |

## CONVENTIONS (THIS DIR)
- **结构化反模式章节**：`init-lua.md` §7（常见错误）、`keymaps.md` §8（常见反模式）、`autocmds.md` §4（augroup）
- ❌ 标记坏写法 → ✅ 标记好写法（代码对比格式）
- 比其他目录更强调「为什么」（不仅是「怎么做」）
- 所有外部链接使用 GitHub permalink（固定 commit SHA）

## ANTI-PATTERNS
- autocommands 无 group 参数（会导致 re-source 时重复）
- 省略 keymap 的 `desc` 属性
- init.lua 中包含大量配置（应拆分到各子模块）
