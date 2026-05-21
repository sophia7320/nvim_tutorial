# init.lua — Neovim 配置入口

> **TL;DR**: `init.lua` 是 Neovim 启动时第一个执行的配置文件。它应该**保持最小化**，仅负责按正确顺序加载子模块。

---

## 1. 为什么 init.lua 取代 init.vim？

Neovim 0.5+ 支持 `init.lua` 作为配置文件（与 `init.vim` 互斥）。**Lua 的优势**：

| 维度 | Vimscript | Lua |
|------|-----------|-----|
| 速度 | 慢（解释执行） | 快（JIT 编译 + 字节码缓存） |
| 模块化 | `source`（全局污染） | `require()`（干净的作用域隔离） |
| 类型安全 | 无 | 可选（LuaLS 注解） |
| 生态 | 停滞 | 活跃插件均用 Lua |
| 学习成本 | 独特的 DSL | 通用编程语言 |

> 来源：[Neovim Lua 指南](https://neovim.io/doc/user/lua-guide.html)

---

## 2. 最小化 init.lua 模板

```lua
-- ~/.config/nvim/init.lua
-- ============================================================================
-- Neovim 0.11+ 配置入口
-- 原则：保持最小化，仅负责按正确顺序加载子模块
-- ============================================================================

-- 🔥 启用 Lua 字节码缓存（Neovim 0.9+，约 30% 启动速度提升）
vim.loader.enable()

-- ⚠️ 必须在加载任何插件之前设置 Leader 键！
vim.g.mapleader = " "
vim.g.maplocalleader = "\\"

-- 加载顺序很重要：
-- 1. 先设置选项（因为后续模块可能依赖某些选项值）
require("config.options")

-- 2. 再设置按键映射
require("config.keymaps")

-- 3. 加载插件管理器（必须最后，因为 lazy.nvim 需要知道完整的配置）
require("config.lazy")

-- 4. 加载自动命令（在插件之后，因为某些 autocmd 依赖插件）
require("config.autocmds")
```

> 来源：[fransys.io 2026 配置指南](https://fransys.io/en/blog/configure-neovim-lua-lazy) | [rubenhortas 2026 指南](https://rubenhortas.github.io/posts/moden-neovim-ide-lua-guide-2026/)

---

## 3. vim.loader.enable() 详解

**Neovim 0.9+ 内置的 Lua 字节码缓存**。工作原理：

```lua
vim.loader.enable() -- 启用缓存
```

- 首次加载 `.lua` 文件时，Neovim 将其编译为字节码并缓存
- 后续启动直接加载缓存，跳过解析和编译步骤
- **实测效果**：约 30% 启动速度提升（[fransys.io 测量](https://fransys.io/en/blog/configure-neovim-lua-lazy)）

---

## 4. 加载顺序的重要性

错误的加载顺序会导致：

| 错误顺序 | 后果 |
|----------|------|
| 插件管理器在 leader 之前 | Leader 键不会被插件识别 |
| 选项在 keymaps 之前 | 没问题（推荐） |
| autocmds 在插件之前 | 某些 `LspAttach` 等事件在插件加载前触发，映射可能丢失 |
| keymaps 在插件之前 | 全局 keymaps 可能被插件覆盖 |

**LazyVim 的处理方式**（[源码参考](https://github.com/folke/lazyvim/blob/main/lua/lazyvim/config/init.lua)）：
- `options` 立即加载
- `keymaps` 和 `autocmds` **延迟到 `User VeryLazy` 事件后加载**，优化启动速度

```lua
-- LazyVim 的分阶段加载策略（简化版）
function M.load(name)
  if name == "options" then
    require("lazyvim.config.options")
  end
  -- keymaps 和 autocmds 延迟加载
  vim.api.nvim_exec_autocmds("User", { pattern = "LazyVim" .. name, modeline = false })
end
```

---

## 5. Lua 模块加载机制

### 5.1 require() 规则

`lua/` 目录下的文件通过 `require()` 加载：

```text
~/.config/nvim/
└── lua/
    ├── config/
    │   └── options.lua        → require("config.options")
    └── plugins/
        ├── lsp.lua             → require("plugins.lsp")
        └── ui.lua              → require("plugins.ui")
```

**关键规则**：
- 点号 `.` 等于路径分隔符 `/`
- 目录下的 `init.lua` 可以直接通过目录名加载：`require("config")` → `lua/config/init.lua`

### 5.2 安全加载（pcall）

```lua
-- 安全加载可能出错的模块
local ok, mymod = pcall(require, "module_with_error")
if not ok then
  vim.notify("Failed to load module_with_error", vim.log.levels.WARN)
else
  mymod.setup()
end
```

> 来源：[Neovim lua-guide.txt](https://github.com/neovim/neovim/blob/v0.11.4/runtime/doc/lua-guide.txt)

### 5.3 开发时热重载

```lua
-- 清除缓存后重新加载（开发时使用）
package.loaded["config.options"] = nil
require("config.options")

-- 或使用 :luafile 命令直接执行（不缓存）
-- :luafile ~/.config/nvim/lua/config/options.lua
```

---

## 6. 进阶：LazyVim 的加载策略

LazyVim 使用 **分阶段 + 事件驱动** 的加载策略，93 个插件启动仅需 ~11ms：

```lua
-- 阶段 1：立即加载（sync）
vim.loader.enable()
vim.g.mapleader = " "
require("lazyvim.config.options")  -- 选项立即生效

-- 阶段 2：延迟到 Init 事件（pre-ui）
-- 加载 UI 相关设置

-- 阶段 3：延迟到 VeryLazy 事件（post-ui）
-- 加载 keymaps, autocmds, 非关键设置
```

> 来源：[LazyVim 源码](https://github.com/folke/lazyvim/blob/main/lua/lazyvim/config/init.lua)

---

## 7. 常见错误

### ❌ 错误：Leader 键在 lazy.nvim 之后设置

```lua
require("config.lazy") -- 插件已加载
vim.g.mapleader = " "  -- ❌ 太晚了！
```

### ✅ 正确：Leader 键在最前面

```lua
vim.g.mapleader = " "  -- ✅ 最先设置
vim.g.maplocalleader = "\\"
require("config.lazy")
```

### ❌ 错误：init.lua 中包含大量配置

```lua
-- ❌ 反模式：把所有配置塞进 init.lua
vim.opt.number = true
vim.opt.relativenumber = true
vim.keymap.set('n', '<leader>f', ...)
-- ... 500 行 ...
```

### ✅ 正确：拆分为独立模块

```lua
require("config.options")   -- 所有 vim.opt 设置
require("config.keymaps")   -- 所有按键映射
```

---

## 8. 关键参考

| 资源 | URL |
|------|-----|
| Neovim 官方 Lua 指南 | https://neovim.io/doc/user/lua-guide.html |
| Neovim 0.11 新特性 | https://neovim.io/doc/user/news-0.11.html |
| LazyVim 配置入口 | https://github.com/folke/lazyvim/blob/main/lua/lazyvim/config/init.lua |
| kickstart.nvim init.lua | https://github.com/nvim-lua/kickstart.nvim/blob/master/init.lua |
| 2026 模块化配置指南 | https://rubenhortas.github.io/posts/moden-neovim-ide-lua-guide-2026/ |
