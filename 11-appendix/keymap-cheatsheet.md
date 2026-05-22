# 全教程默认键位速查

> **TL;DR**: 本教程中推荐的全部 keymap 汇总。`🔧` = Neovim 内置（无需配置）；`🛠` = 需在配置文件中手动定义。

## 全局键位

### 窗口导航

| 按键 | 功能 | 来源 |
|------|------|:---:|
| `<C-h/j/k/l>` | 移动到左/下/上/右窗口 | 🛠 |
| `<C-Up/Down>` | 增加/减少窗口高度 | 🛠 |
| `<C-Left/Right>` | 减少/增加窗口宽度 | 🛠 |

### 搜索

| 按键 | 功能 | 来源 |
|------|------|:---:|
| `<Esc>` | 清除搜索高亮 | 🛠 |

### 标签页

| 按键 | 功能 | 来源 |
|------|------|:---:|
| `<leader><tab>l/f/n/p` | 最后/第一/下/上一个标签 | 🛠 |
| `<leader><tab>c/d` | 新建/关闭标签 | 🛠 |

### 常规

| 按键 | 功能 | 来源 |
|------|------|:---:|
| `<C-s>` | 保存 | 🛠 |
| `<leader>qq` | 退出 | 🛠 |
| `<Esc><Esc>` (Terminal) | 退出终端模式 | 🛠 |

---

## 插件快捷键

### Telescope (查找)

| 按键 | 功能 | 来源 |
|------|------|:---:|
| `<leader>ff` | 查找文件 | 🛠 |
| `<leader>fg` | 实时搜索 (grep) | 🛠 |
| `<leader>fb` | 缓冲区列表 | 🛠 |
| `<leader>fh` | 帮助文档 | 🛠 |
| `<leader>fo` | 最近文件 | 🛠 |
| `<leader>fk` | 按键映射 | 🛠 |
| `<leader>fc` | 命令列表 | 🛠 |
| `<leader>fs` | 文档符号 (LSP) | 🛠 |
| `<leader>fS` | 工作区符号 (LSP) | 🛠 |
| `<leader>fd` | 诊断列表 | 🛠 |

### LSP

| 按键 | 模式 | 功能 | 来源 |
|------|------|------|------|
| `gd` | n | 跳转到定义 | 🔧 内置 |
| `gD` | n | 跳转到声明 | 🛠 自定义 |
| `grn` | n | 重命名 | 🔧 内置 |
| `gra` | n/v | 代码操作 | 🔧 内置 |
| `grr` | n | 查找引用 | 🔧 内置 |
| `gri` | n | 查找实现 | 🔧 内置 |
| `grt` | n | 类型定义 | 🔧 内置 |
| `grx` | n | 执行 CodeLens | 🔧 内置 |
| `gO` | n | 文档符号 | 🔧 内置 |
| `K` | n | 悬停文档 | 🔧 内置 |
| `<C-S>` | i | 签名帮助 | 🔧 内置 |
| `gl` | n | 行诊断详情 | 🛠 自定义 |
| `[d` / `]d` | n | 上/下一个诊断 | 🔧 内置 |
| `<leader>th` | n | 切换 Inlay Hints | 🛠 自定义 |

> 🔧 = Neovim 0.11 内置，无需配置。🛠 = 需在配置文件中手动定义（位置见对应教程章节）。

### Git (gitsigns)

| 按键 | 功能 | 来源 |
|------|------|:---:|
| `]h` / `[h` | 下/上一个 hunk | 🛠 |
| `<leader>hs` | 暂存 hunk | 🛠 |
| `<leader>hr` | 重置 hunk | 🛠 |
| `<leader>hp` | 预览 hunk | 🛠 |
| `<leader>hb` | 行 Blame | 🛠 |
| `<leader>gs` | Neogit 状态 | 🛠 |
| `<leader>gd` | Diffview 差异 | 🛠 |
| `<leader>gh` | Diffview 文件历史 | 🛠 |

### C++ 构建 (cmake-tools)

| 按键 | 功能 | 来源 |
|------|------|:---:|
| `<leader>bc` | CMake Configure | 🛠 |
| `<leader>bb` | CMake Build | 🛠 |
| `<leader>br` | Run Target | 🛠 |
| `<leader>bd` | Debug Target | 🛠 |
| `<leader>bt` | Run Tests | 🛠 |

### 调试 (nvim-dap)

| 按键 | 功能 | 来源 |
|------|------|:---:|
| `<F5>` | 启动/继续 | 🛠 |
| `<F9>` | 切换断点 | 🛠 |
| `<F10>` | 逐过程 | 🛠 |
| `<F11>` | 逐语句 | 🛠 |
| `<F12>` | 跳出 | 🛠 |
| `<leader>dr` | 打开 REPL | 🛠 |
| `<leader>du` | 切换调试 UI | 🛠 |

### 文件管理

| 按键 | 功能 | 来源 |
|------|------|:---:|
| `<leader>e` | neo-tree 切换 | 🛠 |
| `<leader>E` | neo-tree 定位当前文件 | 🛠 |
| `-` | oil.nvim 打开 | 🛠 |

### 其他

| 插件 | 按键 | 功能 | 来源 |
|------|------|------|:---:|
| blink.cmp | `<CR>` | 接受补全 (enter preset) | 🛠 |
| blink.cmp | `<Tab>` | 跳转片段占位符 | 🛠 |
| flash | `s` | 快速跳转 | 🛠 |
| flash | `S` | Treesitter 跳转 | 🛠 |
| which-key | `<leader>` | 弹出按键菜单 | 🛠 |
| toggleterm | `<leader>tt` | 打开终端 | 🛠 |
| codecompanion | `<leader>aa` | AI 内联助手 | 🛠 |
| codecompanion | `<leader>ac` | AI 聊天 | 🛠 |
| conform | `<leader>f` | 手动格式化 | 🛠 |
