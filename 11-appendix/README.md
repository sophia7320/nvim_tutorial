# 附录 — 生态地图、排错指南、参考资源

---

## 1. Neovim 插件生态地图 (2025-2026)

```
                          ┌─────────────────┐
                          │   Neovim 0.11+  │
                          └────────┬────────┘
                                   │
          ┌────────────────────────┼────────────────────────┐
          │                        │                        │
   ┌──────▼──────┐          ┌──────▼──────┐          ┌──────▼──────┐
   │  插件管理器  │          │   核心配置   │          │   LSP 客户端 │
   │ lazy.nvim   │          │ options.lua │          │ vim.lsp.*   │
   └──────┬──────┘          │ keymaps.lua │          └──────┬──────┘
          │                 │ autocmds.lua│                 │
          │                 └─────────────┘                 │
          │                                                │
   ┌──────▼─────────────────────────────────────────────────▼──────────┐
   │                         插件层                                     │
   │                                                                    │
   │  ┌───────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
   │  │ 补全系统   │ │ Treesitter│ │   UI     │ │  编辑    │           │
   │  │ blink.cmp │ │ nvim-    │ │ catppuccin│ │ telescope│           │
   │  │ nvim-cmp  │ │ treesitter│ │ lualine  │ │ which-key│           │
   │  │ LuaSnip   │ │ textobj  │ │ bufferline│ │ flash    │           │
   │  └───────────┘ │ context  │ │ dashboard│ │ mini.nvim│           │
   │                └──────────┘ └──────────┘ └──────────┘           │
   │                                                                    │
   │  ┌───────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
   │  │   DAP     │ │   Git    │ │    AI    │ │ 语言支持 │           │
   │  │ nvim-dap  │ │ gitsigns │ │ copilot  │ │ mason    │           │
   │  │ dap-ui    │ │ neogit   │ │ codecomp │ │ lspconfig│           │
   │  │ dap-vtext │ │ diffview │ │ avante   │ │ conform  │           │
   │  └───────────┘ └──────────┘ └──────────┘ │ nvim-lint│           │
   │                                           └──────────┘           │
   └──────────────────────────────────────────────────────────────────┘
```

---

## 2. 故障排查速查表

### 启动问题

| 问题 | 诊断命令 | 解决方案 |
|------|----------|----------|
| 启动报错 | `nvim --startuptime /tmp/startup.log` | 查看日志，确认错误模块 |
| 插件未加载 | `:Lazy` 查看状态 | `:Lazy sync` |
| 启动慢 | `:Lazy profile` | 优化懒加载配置 |

### LSP 问题

| 问题 | 诊断命令 | 解决方案 |
|------|----------|----------|
| LSP 未启动 | `:LspInfo` | 检查是否安装了服务器 `:Mason` |
| 诊断不显示 | `:checkhealth lsp` | 检查 vim.diagnostic.config 配置 |
| 补全不工作 | 检查 blink.cmp/cmp 状态 | 确认 source 已正确配置 |
| 格式化不生效 | `:ConformInfo` | 检查 formatter 是否安装 |

### Treesitter 问题

| 问题 | 诊断命令 | 解决方案 |
|------|----------|----------|
| 高亮异常 | `:checkhealth nvim-treesitter` | `:TSUpdate` 更新 parser |
| Parser 编译失败 | `:TSInstallSync lua` | 安装 build-essential |

### 键位问题

| 问题 | 诊断命令 | 解决方案 |
|------|----------|----------|
| 快捷键不生效 | `:verbose map <leader>ff` | 检查是否有冲突映射 |
| which-key 不显示 | `:WhichKey <leader>` | 确保映射有 `desc` 属性 |

### Git 问题

| 问题 | 诊断命令 | 解决方案 |
|------|----------|----------|
| gitsigns 无标记 | `:Gitsigns` 命令是否存在 | `:Lazy load gitsigns.nvim` |
| Neogit 报错 | `:checkhealth neogit` | 确认 git 在 PATH 中 |

---

## 3. 健康检查综合命令

```vim
:checkhealth              " 全部健康检查
:checkhealth lsp          " LSP 诊断
:checkhealth nvim-treesitter
:checkhealth provider     " Python/Node/Ruby Provider
:Lazy health              " 插件健康检查
```

---

## 4. 推荐参考资源

### 官方文档

| 资源 | URL |
|------|-----|
| Neovim 官方文档 | https://neovim.io/doc/ |
| Neovim Lua 指南 | https://neovim.io/doc/user/lua-guide.html |
| Neovim LSP 文档 | https://neovim.io/doc/user/lsp.html |
| lazy.nvim 文档 | https://lazy.folke.io/ |
| blink.cmp 文档 | https://cmp.saghen.dev/ |
| mini.nvim 文档 | https://nvim-mini.org/mini.nvim/ |
| CodeCompanion 文档 | https://codecompanion.olimorris.dev/ |

### 参考配置

| 资源 | URL | 特点 |
|------|-----|------|
| kickstart.nvim | https://github.com/nvim-lua/kickstart.nvim | 官方入门，单文件 |
| LazyVim | https://github.com/LazyVim/LazyVim | 最流行的发行版 |
| NvChad | https://github.com/NvChad/NvChad | 社区驱动的发行版 |
| AstroNvim | https://github.com/AstroNvim/AstroNvim | 功能齐全的发行版 |

### 社区与教程

| 资源 | URL |
|------|-----|
| Neovim Reddit | https://reddit.com/r/neovim |
| Neovim Discourse | https://neovim.discourse.group/ |
| This Week in Neovim | https://this-week-in-neovim.org/ |
| Awesome Neovim | https://github.com/rockerBOO/awesome-neovim |
| Neovim 最佳实践 | https://github.com/lumen-oss/nvim-best-practices |

### 2025-2026 年推荐阅读

| 文章 | URL |
|------|-----|
| 2026 模块化 Neovim 指南 | https://rubenhortas.github.io/posts/moden-neovim-ide-lua-guide-2026/ |
| Neovim from scratch 2026 | https://fransys.io/en/blog/configure-neovim-lua-lazy |
| Modern Neovim setup 2026 | https://devtoolsguide.com/neovim-setup-2026/ |
| Neovim 0.11 LSP 指南 | https://zenn.dev/pandanoir/articles/4736924f5ecc72 |
| AI Neovim 插件对比 2026 | https://samuellawrentz.com/blog/neovim-ai-plugins-avante-codecompanion/ |
| blink.cmp vs nvim-cmp | https://gist.github.com/saghen/e731f6f6e30a4c01f6bc7cdaa389d463 |

---

## 5. 环境变量速查

```lua
vim.fn.stdpath("config") -- ~/.config/nvim
vim.fn.stdpath("data")   -- ~/.local/share/nvim
vim.fn.stdpath("state")  -- ~/.local/state/nvim
vim.fn.stdpath("cache")  -- ~/.cache/nvim
vim.fn.stdpath("run")    -- /run/user/1000/nvim.xxx
```
