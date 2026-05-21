# LSP 服务器配置 — nvim-lspconfig + Neovim 0.11 原生 API 深度解析

> **TL;DR**: Neovim 0.11 起，LSP 配置由 `vim.lsp.config()` + `vim.lsp.enable()` 接管。`nvim-lspconfig` 不再作为框架使用，而是作为**配置数据源**提供 200+ 服务器的预设值。本章逐行解析完整的 LSP 配置流程。

---

## 1. 范式转变：旧 vs 新

```lua
-- ─── ❌ 旧范式（0.10 及以前，已废弃）───
-- 问题：每个服务器都要手动调用 setup()，且 setup() 既配置又激活
local lspconfig = require("lspconfig")   -- 加载 nvim-lspconfig 框架
lspconfig.lua_ls.setup({                 -- .setup() 同时做配置 + 启动
  settings = { Lua = { ... } }           -- 服务器特定设置
})
lspconfig.ts_ls.setup({})                -- 重复调用每个服务器

-- ─── ✅ 新范式（0.11+）───
-- 优势：配置和激活分离；配置可持久化到 lsp/*.lua 文件
vim.lsp.config("lua_ls", {              -- 仅配置，不激活
  settings = { Lua = { ... } }
})
vim.lsp.config("ts_ls", {})              -- 每个服务器独立配置
vim.lsp.enable({ "lua_ls", "ts_ls" })   -- 批量激活：根据文件类型自动附加
```

> 来源：[nvim-lspconfig README](https://github.com/neovim/nvim-lspconfig/blob/master/README.md) | [Neovim 0.11 news](https://neovim.io/doc/user/news-0.11.html)

---

## 2. 逐行注解：完整 LSP 插件配置

```lua
-- ============================================================================
-- lua/plugins/lsp.lua — 完整的 LSP 插件栈
-- 文件：~/.config/nvim/lua/plugins/lsp.lua
-- ============================================================================

return {
  -- ═══════════════════════════════════════════════════════════════
  -- 组件 1：mason.nvim — 工具安装器
  -- ═══════════════════════════════════════════════════════════════
  {
    -- (1) 短标识符 — lazy.nvim 自动展开为 https://github.com/mason-org/mason.nvim
    "mason-org/mason.nvim",

    -- (2) opts = {} — 自动调用 require("mason").setup({})
    --     空表表示全部使用默认值（UI、PATH 等默认行为已足够好）
    opts = {},

    -- (3) lazy 未设置 — 继承全局 defaults.lazy，建议此插件不懒加载
  },

  -- ═══════════════════════════════════════════════════════════════
  -- 组件 2：nvim-lspconfig — LSP 服务器配置数据集合
  -- ═══════════════════════════════════════════════════════════════
  {
    -- (4) "neovim/nvim-lspconfig" — Neovim 官方维护的 LSP 预设配置库
    --     它提供 200+ 服务器的默认 cmd、filetypes、root_markers 等配置
    --     在 0.11+ 中不再作为框架，自动将预设写入 lsp/ 目录供 Neovim 加载
    "neovim/nvim-lspconfig",

    -- (5) dependencies — 确保 mason 先加载（nvim-lspconfig 本身不需要 mason，
    --     但整个 LSP 栈的加载顺序要求 mason 先就位）
    dependencies = { "mason-org/mason.nvim" },

    -- (6) 注意：nvim-lspconfig 在 0.11+ 中几乎不需要 config 函数
    --     它的预设配置会自动被 vim.lsp.enable() 使用
    --     你只需通过 vim.lsp.config() 覆盖需要的部分
  },

  -- ═══════════════════════════════════════════════════════════════
  -- 组件 3：mason-lspconfig.nvim — 桥接层
  -- ═══════════════════════════════════════════════════════════════
  {
    -- (7) mason-lspconfig 的职责：建立 "mason 中的包名" ↔ "LSP 服务器名" 的映射
    --     例如：mason 中的 "lua-language-server" → LSP 中的 "lua_ls"
    "mason-org/mason-lspconfig.nvim",

    -- (8) 依赖两个上游
    dependencies = {
      "mason-org/mason.nvim",          -- 需要 mason 来安装工具
      "neovim/nvim-lspconfig",         -- 需要 lspconfig 的预设配置
    },

    opts = {
      -- (9) ensure_installed — 自动确保这些 LSP 服务器已安装
      --     首次运行 Neovim 时，mason 会自动下载列表中缺失的服务器
      --     格式：使用 LSP 服务器名（不是 mason 包名！）
      --     列表中的项对应 :LspInfo 中显示的服务器名称
      ensure_installed = {
        "lua_ls",        -- Lua 语言服务器（原名 sumneko_lua）
        "ts_ls",         -- TypeScript/JavaScript（原名 tsserver）
        "rust_analyzer", -- Rust 语言服务器
        "pyright",       -- Python 类型检查（Microsoft）
        "gopls",         -- Go 官方语言服务器
        "clangd",        -- C/C++（基于 LLVM/Clang）
        "html",          -- HTML 语言支持
        "cssls",         -- CSS/LESS/SASS（vscode-css-languageserver）
        "jsonls",        -- JSON Schema 支持
      },

      -- (10) automatic_enable — 是否自动调用 vim.lsp.enable() 激活已安装的服务器
      --      true（默认）：mason-lspconfig 安装完成后自动激活服务器
      --      false：你需要手动调用 vim.lsp.enable()
      --     也可以是 table 形式：{ exclude = { "rust_analyzer" } } 排除特定服务器
      automatic_enable = true,

      -- (11) handlers — 可以为每个服务器指定安装后的回调
      --      默认 handler 只需 vim.lsp.enable(server_name)
      --      也可以用此钩子做额外的服务器特定配置
      handlers = {
        -- (11a) lua_ls 专用 handler：将 vim 标记为全局变量
        --      这样 lua-language-server 就不会对 vim.* 报「未定义的全局变量」
        --      automatic_enable = true 会自动调用 vim.lsp.enable()，只需配置即可
        ["lua_ls"] = function()
          vim.lsp.config("lua_ls", {
            settings = {
              Lua = {
                diagnostics = { globals = { "vim" } },
              },
            },
          })
        end,
        -- (11b) 默认 handler：没有专属 handler 的服务器走这里
        --      automatic_enable 已自动处理激活，无需额外操作
      },
    },
  },
}
```

---

## 3. 逐行注解：vim.lsp.config() 服务器特定配置

```lua
-- ============================================================================
-- lua/config/lsp.lua — LSP 服务器特定配置 + 诊断全局配置
-- 文件：~/.config/nvim/lua/config/lsp.lua
-- 在 init.lua 中通过 require("config.lsp") 加载
-- ============================================================================

-- ─── 3.1 全局诊断显示配置 ───────────────────────────────────
-- vim.diagnostic.config() 设置所有 LSP 诊断（错误/警告/提示）的显示方式
vim.diagnostic.config({

  -- (1) virtual_text — 在代码行尾以内联文本方式显示诊断消息
  --     true → 默认格式 "[E] message here"
  --     详细配置子表：
  virtual_text = {
    -- (1a) source = "if_many" — 仅当存在多个诊断来源时才显示来源名（如 [pyright]）
    --       可选值：true（总是显示）| false（从不显示）| "if_many"（多来源时显示）
    source = "if_many",

    -- (1b) prefix = "●" — 诊断消息的前缀字符
    --       可以设为自定义字符串，如 "→ " 或使用图标 " "
    prefix = "●",

    -- (1c) spacing = 4 — 消息与代码之间的空格数
    spacing = 4,

    -- (1d) current_line = false（默认）— 不在当前行显示（避免干扰正在输入的代码）
    --       设为 true 会在当前行也显示诊断
  },

  -- (2) signs — sign column（左侧符号列）中的诊断图标
  --     Neovim 0.11+ 使用此方式配置，旧式 vim.fn.sign_define() 已不宜使用
  signs = {
    -- (2a) text — 使用 Unicode/Nerd Font 图标表示四种严重级别
    --      vim.diagnostic.severity 的值：ERROR=1, WARN=2, INFO=3, HINT=4
    text = {
      [vim.diagnostic.severity.ERROR] = " ",   -- nf-fa-times_circle (红色叉圈)
      [vim.diagnostic.severity.WARN]  = " ",   -- nf-fa-warning (黄色三角)
      [vim.diagnostic.severity.INFO]  = " ",   -- nf-fa-info_circle (蓝色信息)
      [vim.diagnostic.severity.HINT]  = "󰌵 ",   -- nf-md-lightbulb_on (灯泡提示)
    },
    -- (2b) numhl — 为行号添加高亮颜色（对应诊断级别）
    --       可选，此处省略以保持简洁
  },

  -- (3) underline — 在诊断位置显示下划线
  --     true：默认波浪线样式
  --     可设为 table 精确控制：{ severity = { min = WARN } } 仅警告及以上
  underline = true,

  -- (4) update_in_insert — 插入模式下是否更新诊断
  --     false（推荐）：插入模式不刷新诊断，避免性能下降和视觉干扰
  --     true：每次输入都刷新（可能导致卡顿）
  update_in_insert = false,

  -- (5) severity_sort — 按严重级别排序诊断
  --     true：错误在最前 → 警告 → 信息 → 提示
  --     这确保最严重的问题最容易被看到
  severity_sort = true,

  -- (6) float — 浮动诊断窗口的样式（:lua vim.diagnostic.open_float() 时使用）
  float = {
    -- (6a) border = "rounded" — 圆角边框
    --       可选："none" | "single" | "double" | "rounded" | "solid" | "shadow"
    border = "rounded",

    -- (6b) source = true — 在浮动窗口中显示诊断来源（如 [pyright]）
    source = true,

    -- (6c) header — 浮动窗口标题行
    --       空字符串 → 无标题
    --       可设为 "Diagnostics:" 等自定义文字
    header = "",

    -- (6d) prefix — 消息前缀（类似 virtual_text.prefix）
    prefix = "",
  },
})

-- ─── 3.2 Lua 语言服务器特定配置 ──────────────────────────────
vim.lsp.config("lua_ls", {
  -- (7) cmd — 启动语言服务器的命令
  --     对于 mason 安装的服务器，只需提供命令名，因为 mason 的 bin/ 已在 PATH 中
  --     也可指定完整路径：vim.fn.stdpath("data") .. "/mason/bin/lua-language-server"
  cmd = { "lua-language-server" },

  -- (8) filetypes — 此 LSP 服务器响应的文件类型列表
  --     只有打开这些类型的文件时，服务器才会自动附加
  filetypes = { "lua" },

  -- (9) root_markers — 确定项目根目录的标记文件列表
  --     Neovim 从文件所在目录开始向上查找，遇到任一标记文件即停止
  --     .luarc.json/.luarc.jsonc 是 lua-language-server 的专用配置文件
  --     .git 是最通用的回退选项
  root_markers = { ".luarc.json", ".luarc.jsonc", ".git" },

  -- (10) settings — 传递给语言服务器的 JSON-like 配置
  --      这是一个嵌套 Lua table，会被转换为 JSON 发送给 LSP 服务器
  --      每个服务器的 settings 结构完全不同，需查阅其官方文档
  settings = {
    -- (10a) Lua 顶层键 — lua-language-server 的所有配置都在此键下
    Lua = {

      -- (10b) runtime — 运行时环境配置
      runtime = {
        -- (10c) version = "LuaJIT" — 告诉服务器使用 LuaJIT 的语法规则
        --       Neovim 内嵌 LuaJIT，所以这里必须匹配
        --       如果设为 "Lua 5.1"，服务器会误报一些 LuaJIT 独有的语法
        version = "LuaJIT",
      },

      -- (10d) diagnostics — 诊断规则配置
      diagnostics = {
        -- (10e) globals — 声明全局变量，避免服务器报告 "未定义的全局变量"
        --       因为 Neovim 的 Lua 环境中 vim 是预定义的全局变量
        --       如果你还用了其他全局库（如 "hs" for hammerspoon），也加在这里
        globals = { "vim" },
      },

      -- (10f) workspace — 工作区配置
      workspace = {
        -- (10g) checkThirdParty = false — 禁用第三方代码检查
        --       设为 false 可大幅提升性能，因为服务器不会去分析
        --       node_modules、.git 等目录中的 Lua 代码
        checkThirdParty = false,

        -- (10h) library — 告诉服务器哪些目录包含可用的 Lua 库
        --       vim.env.VIMRUNTIME 是 Neovim 的运行库路径
        --       (~/.local/share/nvim/runtime 或 /usr/share/nvim/runtime)
        --       添加后，vim.* 等 API 的补全和类型检查才能正常工作
        library = { vim.env.VIMRUNTIME },
      },

      -- (10i) telemetry — 遥测/使用数据上报配置
      telemetry = {
        -- (10j) enable = false — 禁用向服务器开发者发送使用数据
        --       隐私敏感用户必须设为 false
        enable = false,
      },
    },
  },
})

-- ─── 3.3 批量激活所有服务器 ──────────────────────────────────
-- vim.lsp.enable() 根据文件类型将配置好的服务器附加到缓冲区
-- 传入服务器名列表来批量激活（名称必须与 vim.lsp.config 的调用一致）
vim.lsp.enable({
  "lua_ls",
  "ts_ls",
  "rust_analyzer",
  "pyright",
  "gopls",
  "clangd",
})
```

---

## 4. 逐行注解：LspAttach 回调中的按键映射

```lua
-- ============================================================================
-- 接续 lua/config/lsp.lua — LspAttach 自动命令
-- ============================================================================

-- (11) nvim_create_autocmd — 创建自动命令，在 LSP 每次附加到缓冲区时执行回调
vim.api.nvim_create_autocmd("LspAttach", {
  -- (12) group — 自动命令组，用于批量管理和防止重复
  --       nvim_create_augroup 创建新组，{ clear = true } 确保每次执行前清除旧定义
  group = vim.api.nvim_create_augroup("user_lsp_attach", { clear = true }),

  -- (13) callback — 事件触发时执行的函数
  --       event 参数包含 .data.client_id（LSP 客户端 ID）和 .buf（缓冲区编号）
  callback = function(event)
    -- (14) vim.lsp.get_client_by_id() — 通过客户端 ID 获取 client 对象
    --      用于后续检查客户端的能力（supports_method）
    local client = vim.lsp.get_client_by_id(event.data.client_id)

    -- (15) 缓冲区编号 — 用于设置缓冲区本地的按键映射（buffer = bufnr）
    local bufnr = event.buf

    -- ─── 辅助函数：减少重复代码 ─────────────────────────────
    -- (16) 定义本地 map 函数，自动附加 buffer 和 desc 参数
    --      keys: 按键组合（string）
    --      func: 要调用的 LSP 函数
    --      desc: which-key 显示用的描述文字
    --      mode: 映射模式，默认 "n"（Normal 模式）
    local map = function(keys, func, desc, mode)
      mode = mode or "n"
      vim.keymap.set(mode, keys, func,
        { buffer = bufnr, desc = "LSP: " .. desc })  -- 仅当前缓冲区生效
    end

    -- ─── 基础 LSP 映射 ──────────────────────────────────────
    -- 注意：Neovim 0.11 已内置 grn (重命名)、gra (代码操作)、grr (引用)
    --       gd (跳转定义)、K (悬停文档) 等默认映射
    --       以下仅添加额外的/覆盖的映射

    -- (17) gD — 跳转到声明（与 gd 跳转到定义不同）
    --       只在客户端支持 textDocument/declaration 方法时才映射
    --       client:supports_method() 动态检查能力，避免映射无效按键
    if client:supports_method("textDocument/declaration", bufnr) then
      map("gD", vim.lsp.buf.declaration, "[G]oto [D]eclaration")
    end

    -- ─── 诊断映射 ──────────────────────────────────────────

    -- (18) gl — 显示当前行的诊断详情浮动窗口
    --       vim.diagnostic.open_float 显示诊断的完整消息和来源
    map("gl", function()
      vim.diagnostic.open_float({
        border = "rounded",         -- 浮动窗口边界风格
        source = true,              -- 显示诊断来源
        max_width = 80,             -- 最大窗口宽度（字符数）
        max_height = 20,            -- 最大窗口高度（行数）
        scope = "cursor",           -- 仅显示光标位置的诊断
      })
    end, "Line Diagnostics")

    -- (19) [d — 跳转到上一个诊断
    --      使用 vim.diagnostic.jump()，count = -1 表示向后跳转
    map("[d", function()
      vim.diagnostic.jump({ count = -1, float = true })
    end, "Previous Diagnostic")

    -- (20) ]d — 跳转到下一个诊断
    --      count = 1 表示向前跳转，float = true 自动显示浮动窗口
    map("]d", function()
      vim.diagnostic.jump({ count = 1, float = true })
    end, "Next Diagnostic")

    -- ─── Inlay Hints（0.10+ 特性）──────────────────────────

    -- (21) 检查客户端是否支持 textDocument/inlayHint 方法
    if client:supports_method("textDocument/inlayHint", bufnr) then
      -- (22) 为该缓冲区启用 inlay hints
      --      inlay hints 在代码中显示参数名、类型推断等辅助信息
      vim.lsp.inlay_hint.enable(true, { bufnr = bufnr })

      -- (23) <leader>th — 切换 inlay hints 的显示/隐藏
      map("<leader>th", function()
        -- vim.lsp.inlay_hint.is_enabled() 检查当前缓冲区是否已启用
        -- 取反后传给 enable() 实现切换
        vim.lsp.inlay_hint.enable(
          not vim.lsp.inlay_hint.is_enabled({ bufnr = bufnr }),
          { bufnr = bufnr }
        )
      end, "[T]oggle Inlay [H]ints")
    end
  end,
})
```

---

## 5. 关键参考

| 资源 | URL |
|------|-----|
| Neovim LSP 官方文档 | https://neovim.io/doc/user/lsp.html |
| Neovim 诊断文档 | https://neovim.io/doc/user/diagnostic.html |
| nvim-lspconfig | https://github.com/neovim/nvim-lspconfig |
| nvim-lspconfig 0.11+ 迁移说明 | https://github.com/neovim/nvim-lspconfig/blob/master/README.md |
| mason-lspconfig.nvim | https://github.com/mason-org/mason-lspconfig.nvim |
