# Mason.nvim — LSP 工具安装器深度解析

> **TL;DR**: `mason.nvim` 是 Neovim 生态的便携式包管理器，负责安装/管理 LSP 服务器、DAP 适配器、Linter、Formatter 等**外部命令行工具**。它不配置这些工具——只管下载和管理。

---

## 1. mason.nvim 到底是什么？

mason.nvim 解决的核心问题：**Neovim 本身不带 python 的 linter、rust 的编译器、TypeScript 的语言服务器**。传统做法是手动 `npm install -g` / `pip install` / `cargo install`，导致：

- 系统全局 PATH 被污染
- 不同项目需要不同版本
- 换机器后需要重新安装所有工具

mason.nvim 将所有工具**隔离安装在 Neovim 的数据目录**中（`~/.local/share/nvim/mason/`），不污染系统环境。

> 来源：[mason.nvim README](https://github.com/mason-org/mason.nvim)

---

## 2. 逐行注解：最小化配置

```lua
-- ============================================================================
-- lua/plugins/lsp.lua — mason.nvim 配置块
-- 文件：~/.config/nvim/lua/plugins/lsp.lua
-- ============================================================================

return {
  {
    -- (1) 插件标识符：GitHub 仓库的 "owner/repo" 格式
    --     lazy.nvim 会自动从 https://github.com/mason-org/mason.nvim 克隆
    "mason-org/mason.nvim",

    -- (2) lazy = nil（未设置）= 使用 lazy.nvim 全局默认值
    --     建议 mason.nvim 不要懒加载，因为其他插件（lspconfig/conform/lint）
    --     在启动时就需要通过 mason 检查已安装的工具
    --     如果全局 defaults.lazy = true，此处需显式设为 lazy = false

    -- (3) opts = {} — lazy.nvim 的约定：
    --     设置 opts 后，lazy.nvim 会自动推断主模块名（此处为 "mason"），
    --     并在插件加载时自动执行 require("mason").setup(opts)
    --     等价于手动写：config = function(_, opts) require("mason").setup(opts) end
    opts = {

      -- (3a) ui 子表 — 控制 Mason 的图形界面（:Mason 命令打开的 UI）
      ui = {

        -- (3b) icons 子表 — 自定义 Mason UI 中的图标
        --      需要 Nerd Font 才能正确显示这些 Unicode 图标
        icons = {

          -- (3c) package_installed — 已安装工具的状态图标
          --      "✓" (U+2713) = 对勾，表示该工具已成功安装
          package_installed = "✓",

          -- (3d) package_pending — 正在安装/队列中的图标
          --      "➜" (U+279C) = 箭头，表示安装进行中
          package_pending = "➜",

          -- (3e) package_uninstalled — 未安装工具的图标
          --      "✗" (U+2717) = 叉号，表示尚未安装
          package_uninstalled = "✗",
        },
      },
    },
    -- (4) 注意：opts 只包含 UI 配置。Mason 的核心行为（安装路径等）
    --     使用默认值即可满足 99% 的场景，无需额外配置
  },
}
```

**配置效果**：执行 `:Mason` 后打开图形界面，可以看到所有可用工具的列表，按 `i` 安装，按 `X` 卸载，按 `u` 更新。

---

## 3. 逐行注解：高级配置

```lua
-- ============================================================================
-- lua/plugins/lsp.lua — mason.nvim 完整配置
-- ============================================================================

return {
  {
    "mason-org/mason.nvim",

    -- (1) cmd = "Mason" — 命令驱动延迟加载
    --     只有第一次执行 :Mason 命令时才会加载整个 mason.nvim 插件
    --     但注意：如果其他插件（如 mason-lspconfig）依赖 mason 的 API，
    --     则不应懒加载。此处仅适用于不依赖 mason API 的极简场景
    cmd = "Mason",

    opts = {
      -- (2) ui 配置块（同最小配置）
      ui = {
        -- (2a) border = "rounded" — Mason UI 弹窗使用圆角边框
        --      可选值："none" | "single" | "double" | "rounded" | "solid" | "shadow"
        border = "rounded",

        -- (2b) icons 配置块
        icons = {
          package_installed = "✓",
          package_pending = "➜",
          package_uninstalled = "✗",
        },

        -- (2c) keymaps — Mason UI 内的自定义快捷键
        --     以下键位均为 Mason 默认值，可整块省略
        keymaps = {
          -- (2d) toggle_help = "?" — 按 ? 键切换帮助信息显示
          toggle_help = "?",

          -- (2e) install_package = "i" — 光标移到某工具上按 i 安装 [默认值]
          install_package = "i",

          -- (2f) update_package = "u" — 按 u 更新光标所在工具 [默认值]
          update_package = "u",

          -- (2g) check_package_version = "c" — 按 c 检查新版本 [默认值]
          check_package_version = "c",

          -- (2h) update_all_packages = "U" — 按大U 更新全部工具 [默认值]
          update_all_packages = "U",

          -- (2i) check_outdated_packages = "C" — 按大C 检查所有过时工具 [默认值]
          check_outdated_packages = "C",

          -- (2j) uninstall_package = "X" — 按大X 卸载光标所在工具 [默认值]
          uninstall_package = "X",

          -- (2k) cancel_installation = "<C-c>" — Ctrl+C 取消当前安装 [默认值]
          cancel_installation = "<C-c>",

          -- (2l) apply_language_filter = "<C-f>" — Ctrl+F 按语言筛选 [默认值]
          apply_language_filter = "<C-f>",
        },
      },

      -- (3) PATH 配置 — Mason 安装的工具如何暴露给 Neovim
      --     默认值 "prepend" 表示 Mason 的 bin 目录优先于系统 PATH
      --     可选值："prepend"（推荐）| "append" | "skip"（不修改 PATH）
      --     "prepend" 确保 Mason 安装的 lua-language-server 优先于系统全局安装的版本
      PATH = "prepend",

      -- (4) max_concurrent_installers — 同时最多安装几个工具
      --     默认 4，如果你网络好可以调高加快批量安装
      max_concurrent_installers = 4,

      -- (5) registries — Mason 的软件源（registry）列表
      --     默认使用官方的 GitHub registry：https://github.com/mason-org/mason-registry
      --     可以添加自定义 registry 来安装非官方收录的工具
      registries = {
        -- (5a) 官方 registry：包含 300+ 工具的安装脚本和元数据
        "github:mason-org/mason-registry",
      },

      -- (6) providers — 工具提供者配置
      --     Mason 支持从多种来源安装工具（GitHub Releases、npm、pip 等）
      providers = {
        -- (6a) 默认启用所有 provider，你可以禁用不用的来加速启动
        -- "github" provider — 从 GitHub Release 下载预编译二进制（最常用）
        -- "npm" provider — 从 npm registry 安装 Node.js 工具（如 typescript-language-server）
        -- "pip3" provider — 从 PyPI 安装 Python 工具（如 debugpy）
        -- "go" provider — 从 Go 模块安装 Go 工具（如 gopls）
        -- "cargo" provider — 从 crates.io 安装 Rust 工具（如 stylua）
        -- "gem" provider — 从 RubyGems 安装 Ruby 工具
        -- "composer" provider — 从 Packagist 安装 PHP 工具
      },
    },
  },
}
```

---

## 4. Mason 常用命令

| 命令 | 说明 |
|------|------|
| `:Mason` | 打开 Mason 管理面板 |
| `:MasonInstall <package>` | 安装指定工具（如 `:MasonInstall lua-language-server`） |
| `:MasonUninstall <package>` | 卸载工具 |
| `:MasonUpdate` | 更新所有已安装工具 |
| `:MasonLog` | 查看安装日志（排查安装失败） |
| `:MasonToolsInstall` | 批量安装 ensure_installed 中声明的缺失工具 |
| `:MasonToolsUpdate` | 更新 ensure_installed 中声明的所有工具 |
| `:MasonToolsClean` | 卸载不在 ensure_installed 中的工具 |

---

## 5. 工具安装路径

mason.nvim 将所有工具安装在 Neovim 数据目录下：

```text
~/.local/share/nvim/mason/
├── bin/               # 可执行文件的符号链接
│   ├── lua-language-server  → ../packages/lua-language-server/...
│   ├── stylua               → ../packages/stylua/...
│   └── ...
├── packages/          # 各工具的完整安装目录
│   ├── lua-language-server/
│   ├── stylua/
│   └── ...
└── registry.json      # 本地 registry 缓存
```

**关键**：`bin/` 目录会被自动添加到 Neovim 的 PATH 中，所以 `vim.lsp.config()` 可以直接通过命令名启动服务器。

---

## 6. 与其他插件的协作

```text
mason.nvim (安装工具)
    │
    ├──→ mason-tool-installer.nvim (自动确保 formatter/linter/DAP 等所有工具已安装)
    │
    ├──→ mason-lspconfig.nvim (桥接：自动为 LSP 服务器生成 vim.lsp.enable() 调用)
    │
    ├──→ mason-nvim-dap.nvim  (桥接：自动配置 nvim-dap 适配器)
    │
    └──→ conform.nvim / nvim-lint (直接调用 Mason 安装的 formatter/linter 命令)
```

---

## 7. mason-tool-installer.nvim — 自动安装所有 Mason 工具

### 7.1 为什么需要它？

`mason-lspconfig.nvim` 的 `ensure_installed` 只能自动安装 **LSP 服务器**。但你还需要 formatter（stylua、prettierd）、linter（eslint_d、luacheck）、DAP 适配器（debugpy、codelldb）等。

`mason-tool-installer.nvim` 接管所有 Mason 工具的自动安装——无论类型。**换新机器后打开 Neovim，所有工具自动就位。**

> ╔══════════════════════════════════════════════╗
> ║ mason-lspconfig 的 ensure_installed 详见：     ║
> ║ → lspconfig.md §2 组件 3                       ║
> ╚══════════════════════════════════════════════╝

### 7.2 最小配置（开箱即用）

```lua
-- lua/plugins/lsp.lua — mason-tool-installer 配置块
return {
  {
    -- (1) 插件标识符
    "WhoIsSethDaniel/mason-tool-installer.nvim",

    -- (2) dependencies — 依赖 mason.nvim（必须先安装）
    dependencies = { "mason-org/mason.nvim" },

    -- (3) opts — 最小配置：只需声明你需要哪些工具
    --     mason-tool-installer 会在 Neovim 启动时自动检查并安装缺失的工具
    --     所有工具名与 Mason Registry 中的包名一致（:Mason 里看到的名称）
    --     ⚠️ 这是你唯一需要关心的配置——告诉它你要什么，其余全自动
    opts = {
      ensure_installed = {
        -- LSP 服务器
        "lua-language-server",  -- Lua
        "rust-analyzer",        -- Rust
        "pyright",              -- Python
        "clangd",               -- C/C++

        -- Formatter
        "stylua",               -- Lua 格式化
        "prettierd",            -- JS/TS/CSS/HTML
        "clang-format",         -- C/C++

        -- Linter
        "eslint_d",             -- JS/TS
        "luacheck",             -- Lua
        "clang-tidy",           -- C/C++

        -- DAP 调试器
        "debugpy",              -- Python
        "codelldb",             -- C/C++/Rust
      },
    },
  },
}
```

> 💡 **提示**：使用 mason-tool-installer 后，`mason-lspconfig` 的 `ensure_installed` 可以移除——由前者统一管理所有工具。

### 7.3 高级选项（按需开启）

```lua
-- (4) 完整配置——仅在默认行为不够时使用
opts = {
  -- (4a) ensure_installed — 要确保安装的工具列表（🟡 必需，无默认值）
  ensure_installed = { "stylua", "eslint_d", ... },

  -- (4b) auto_update = false — 启动时自动更新已安装工具
  --      默认 false。开启后每次启动检查更新，略慢但省心
  auto_update = false,

  -- (4c) run_on_start = true — 启动时立即检查安装
  --      默认 true。设为 false 则需要手动 :MasonToolsInstall
  run_on_start = true,

  -- (4d) start_delay = 3000 — 延迟启动检查（ms），默认 3000
  --      调大可避免与其他启动插件抢占资源
  start_delay = 3000,

  -- (4e) debounce_hours = 24 — 自动更新间隔（小时）
  --      配合 auto_update = true，每 24h 检查一次更新
  debounce_hours = 24,
}
```

| 选项 | 类型 | 默认值 | 必需？ | 说明 |
|------|------|--------|:---:|------|
| `ensure_installed` | `string[]` | `{}` | 🟡 **是** | 你要自动安装哪些工具 |
| `auto_update` | `boolean` | `false` | 🟢 否 | 启动时自动更新 |
| `run_on_start` | `boolean` | `true` | 🟢 否 | 启动时立即检查 |
| `start_delay` | `number` | `3000` | 🟢 否 | 延迟启动检查（ms） |
| `debounce_hours` | `number` | `24` | 🟢 否 | 自动更新最小间隔 |

---

## 8. 关键参考

| 资源 | URL |
|------|-----|
| mason.nvim README | https://github.com/mason-org/mason.nvim |
| mason.nvim 官方文档 | https://mason-org.github.io/mason.nvim/ |
| mason-tool-installer.nvim | https://github.com/WhoIsSethDaniel/mason-tool-installer.nvim |
| Mason Registry（所有可用包） | https://mason-registry.dev/ |
