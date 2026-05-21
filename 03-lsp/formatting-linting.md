# Conform.nvim + nvim-lint — 格式化和代码检查深度解析

> **TL;DR**: `conform.nvim` 是现代 Neovim 的**异步格式化器**；`nvim-lint` 是**异步代码检查器**。两者互补——formatter 修风格，linter 查逻辑。本章逐行解析两者的完整配置。

---

## 1. 为什么不用 LSP 自带格式化？

| 维度 | LSP 格式化 | conform.nvim |
|------|-----------|:-----------:|
| 执行方式 | 同步（阻塞编辑器） | **异步**（不阻塞） |
| 多格式化器 | 仅一个 | **链式执行**（如 isort → black） |
| 范围格式化 | 某些服务器不支持 | **全部支持**（含 visual 选区） |
| 对光标/折叠的影响 | 破坏性替换 | **最小化 diff**（保留光标和折叠） |
| 超时控制 | 无 | **可配置 timeout_ms** |

---

## 2. 逐行注解：conform.nvim 完整配置

```lua
-- ============================================================================
-- lua/plugins/lsp.lua — conform.nvim 格式化器
-- 文件：~/.config/nvim/lua/plugins/lsp.lua
-- ============================================================================

return {
  {
    -- (1) 插件标识符
    "stevearc/conform.nvim",

    -- (2) event = { "BufWritePre" } — 在写入文件**之前**触发加载
    --     这意味着：格式化的逻辑在保存前已就绪，但首次加载是惰性的
    --     同时支持 BufWritePre 事件触发的 format_on_save
    event = { "BufWritePre" },

    -- (3) cmd = { "ConformInfo" } — 注册 :ConformInfo 命令
    --     执行此命令时如果插件尚未加载，会自动加载
    --     :ConformInfo 显示当前缓冲区的格式化器信息和可用格式化器列表
    cmd = { "ConformInfo" },

    -- (4) keys — 手动格式化的快捷键
    keys = {
      {
        -- (4a) "<leader>f" — 快捷键组合：leader 键 + f
        "<leader>f",

        -- (4b) mode = { "n", "x" } — Normal + Visual 模式
        --      也可以在 visual 模式下传入 selection 范围
        mode = { "n", "x" },

        -- (4c) 回调函数 — 执行异步格式化
        --      require("conform").format() 启动格式化任务
        --      { async = true } — 不阻塞编辑器 UI
        --      { lsp_format = "fallback" } — 没有专用格式化器时回退到 LSP formatting
        function()
          require("conform").format({
            async = true,            -- 异步执行，编辑器保持响应
            lsp_format = "fallback", -- 备选方案：让 LSP 来格式化
          })
        end,

        -- (4d) desc — which-key 展示文字
        desc = "[F]ormat buffer",
      },
    },

    -- (5) opts — 自动调用 require("conform").setup(opts)
    opts = {

      -- ─── 5.1 formatters_by_ft — 按文件类型指定格式化器 ───
      --      键 = 文件类型（与 :set ft? 输出一致）
      --      值 = 格式化器名称数组，按顺序执行
      formatters_by_ft = {

        -- (5a) lua — 使用 StyLua（Rust 编写的 Lua 格式化器，极快）
        lua = { "stylua" },

        -- (5b) python — 使用 ruff_format（ruff 的子命令，极快）
        --       ruff_format 集成了 isort（import 排序）和 black（代码格式化）
        python = { "ruff_format" },
        -- 备选方案：如果不用 ruff，可改为 { "isort", "black" }
        -- isort 先排序 import，black 再格式化代码

        -- (5c) javascript/typescript — 使用 prettierd（prettier 的 daemon 模式，更快）
        --      stop_after_first = true — 第一个可用即停止，不再尝试列表后续
        --      prettierd 是 prettier 的后台常驻进程版本，省去每次都启动 Node.js 的开销
        javascript = { "prettierd", "prettier", stop_after_first = true },
        typescript = { "prettierd", "prettier", stop_after_first = true },

        -- (5d) go — goimports（先整理 import 再 format）+ gofmt（标准格式化）
        go = { "goimports", "gofmt" },

        -- (5e) rust — rustfmt（Rust 官方格式化器）
        rust = { "rustfmt" },

        -- (5f) 通配符 "*" — 对所有文件类型生效
        --      codespell 是拼写检查器，会自动修正注释中的英文拼写错误
        ["*"] = { "codespell" },

        -- (5g) 特殊键 "_" — "fallback" 文件类型
        --      当以上所有规则都不匹配时使用（例如 .txt 文件）
        --      trim_whitespace — 仅删除行尾空白（内置格式化器，无需安装）
        ["_"] = { "trim_whitespace" },
      },

      -- ─── 5.2 default_format_opts — 默认格式化选项 ─────
      default_format_opts = {
        -- (5h) lsp_format = "fallback" — LSP 格式化的优先级
        --      "never"：从不使用 LSP 格式化
        --      "fallback"：没有专用格式化器时，回退到 LSP formatting
        --      "prefer"：优先使用 LSP 格式化，没有时才用专用格式化器
        lsp_format = "fallback",
      },

      -- ─── 5.3 format_on_save — 保存时自动格式化 ─────────
      format_on_save = {
        -- (5i) timeout_ms = 500 — 最长等待 500 毫秒
        --      超过此时间放弃本次格式化，避免在超大文件上卡死
        --      注意：这是同步等待的时限（format_on_save 是同步执行的）
        timeout_ms = 500,

        -- (5j) lsp_format = "fallback" — 保存时也用专用格式化器为主
        lsp_format = "fallback",
      },

      -- ─── 5.4 formatters — 自定义格式化器参数 ────────────
      formatters = {
        -- (5k) shfmt — 自定义 shfmt（shell 格式化器）的行为
        --      prepend_args — 在默认参数前添加额外参数
        --      "-i" "2" → 缩进 2 个空格
        --      最终命令变为：shfmt -i 2 <file>
        shfmt = {
          prepend_args = { "-i", "2" },
        },
      },
    },

    -- (6) init — 在所有插件加载完成后，且在 config（即 opts 触发的 setup）之前执行
    --     设置 formatexpr 以支持 gq 操作符使用 conform 格式化
    init = function()
      -- (6a) vim.o.formatexpr — Vim 选项，定义 gq 操作符的格式化函数
      --      "v:lua.require'conform'.formatexpr()" →
      --      告诉 Neovim 在 gq/gw 时调用 conform 的 formatexpr 函数
      --      这意味着按 gqip（格式化当前段落）会使用 conform 而不是 LSP
      vim.o.formatexpr = "v:lua.require'conform'.formatexpr()"
    end,
  },
}
```

---

## 3. conform.nvim 常用命令与快捷键

| 命令/快捷键 | 说明 |
|------------|------|
| `:ConformInfo` | 显示当前缓冲区的格式化器状态 |
| `<leader>f` | 手动格式化当前缓冲区 |
| 保存时自动 | 由 `format_on_save` 控制 |
| `gq` 操作符 | 由 `formatexpr` 接管，使用 conform 格式化 |

---

## 4. 逐行注解：nvim-lint 代码检查配置

```lua
-- ============================================================================
-- lua/plugins/lsp.lua — nvim-lint 代码检查器
-- 文件：~/.config/nvim/lua/plugins/lsp.lua
-- ============================================================================

return {
  {
    -- (1) 插件标识符
    "mfussenegger/nvim-lint",

    -- (2) event = { "BufReadPre", "BufNewFile" } — 读取或创建文件时加载
    --     nvim-lint 本身是轻量插件，不需要等到 BufWritePost 才加载
    --     但在 BufReadPre 时就准备好 linter 配置
    event = { "BufReadPre", "BufNewFile" },

    -- (3) 使用 config 而非 opts — 因为 nvim-lint 不使用标准的 setup(opts) 惯例
    config = function()
      -- (4) require("lint") — 加载 nvim-lint 的 Lua 模块
      local lint = require("lint")

      -- ─── 4.1 linters_by_ft — 按文件类型指定 linter ────────
      --      键 = 文件类型．值 = linter 名称数组
      lint.linters_by_ft = {

        -- (4a) python — Ruff 是目前最快的 Python linter（Rust 编写）
        --      mypy 是静态类型检查器（比 pyright 更严格，但检查更多模式）
        --      两者风格互补：ruff 查风格和常见错误，mypy 查类型问题
        python = { "ruff", "mypy" },

        -- (4b) javascript/typescript — eslint_d（eslint 的 daemon 模式）
        --      使用 _d 后缀版本可避免每次启动 Node.js 的开销
        --      daemon 模式：首次启动后保持进程常驻，后续检查仅通信不启动
        javascript = { "eslint_d" },
        typescript = { "eslint_d" },

        -- (4c) markdown — vale（自然语言 prose linter）
        --      检查 Markdown 文档的写作风格，需要配合 .vale.ini 配置使用
        markdown = { "vale" },

        -- (4d) lua — luacheck（Lua 静态分析器）
        lua = { "luacheck" },
      }

      -- ─── 4.2 自动触发 lint — 文件保存后运行 ───────────────
      -- (5) BufWritePost — 文件写入完成后触发
      --     选择此事件（而非 BufWritePre）的原因是：
      --     保存时才运行 lint，读取时不运行——节省 CPU，
      --     且不会在刚打开文件时弹出过时诊断
      vim.api.nvim_create_autocmd({ "BufWritePost" }, {
        desc = "保存后自动运行 linter",
        -- (6) callback — 保存后执行的函数
        callback = function()
          -- (7) lint.try_lint() — 尝试对当前缓冲区运行所有已配置的 linter
          --     如果找不到 linter 可执行文件，不会报错，只是静默跳过
          --     诊断结果会自动注入 vim.diagnostic（与 LSP 诊断共享显示配置）
          lint.try_lint()
        end,
      })

      -- ─── 4.3 按需手动触发 lint ──────────────────────────
      -- 在 init.lua 或 keymaps.lua 中添加：
      -- vim.keymap.set("n", "<leader>cl", function()
      --   require("lint").try_lint()
      -- end, { desc = "手动运行代码检查" })
    end,
  },
}
```

---

## 5. nvim-lint 与 vim.diagnostic 的协作

nvim-lint 本身不负责**显示**诊断——它只负责：

1. 调用外部 linter 命令（如 `ruff check <file>`）
2. 解析输出（stdout/stderr）
3. 将结果注入 `vim.diagnostic` 命名空间

显示则由 `vim.diagnostic.config()` 统一控制（已在 [lspconfig.md](lspconfig.md) 中详细配置）。

这意味着你在 `lspconfig.md` 中配置的虚拟文本、符号列图标、浮动窗口样式对 LSP 诊断和 nvim-lint 诊断**同时生效**——它们共享同一套 UI。

---

## 6. 需要 Mason 安装的工具

以下格式化器和 linter 需要先通过 Mason 安装：

```bash
# 格式化器
:MasonInstall stylua        # Lua
:MasonInstall ruff           # Python (含 ruff_format)
:MasonInstall prettierd      # JS/TS (高效的 daemon 模式)
:MasonInstall goimports      # Go import 整理
rustup component add rustfmt  # Rust（rustfmt 不在 Mason 中，需通过 rustup 安装）

# Linter
:MasonInstall mypy           # Python 类型检查
:MasonInstall eslint_d       # JS/TS daemon linter
:MasonInstall luacheck       # Lua 静态分析
```

---

## 7. 关键参考

| 资源 | URL |
|------|-----|
| conform.nvim | https://github.com/stevearc/conform.nvim |
| conform.nvim recipes | https://github.com/stevearc/conform.nvim/blob/master/doc/recipes.md |
| nvim-lint | https://github.com/mfussenegger/nvim-lint |
| Mason Registry（查找 package） | https://mason-registry.dev/ |
