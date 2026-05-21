# blink.cmp — 现代补全引擎深度解析

> **TL;DR**: `blink.cmp` 是 2025-2026 年 Neovim 补全的新标准。Rust 后端 + 内置源 + typo 容错 + 每键击 0.5-4ms。本章逐行解析完整配置、每个变量、每个源选项。

---

## 1. blink.cmp 架构

```
Trigger (何时触发补全?)
   │
   ▼
Sources (从哪里获取候选项？)
   │  ┌────────┬──────────┬──────────┬──────────┐
   │  │  LSP   │  Buffer  │   Path   │ Snippets │
   │  └────────┴──────────┴──────────┴──────────┘
   ▼
Fuzzy (如何过滤和排序？)
   │  Smith-Waterman 算法 (Rust FFI)
   │  + frecency (频率+最近使用)
   │  + proximity (距光标的远近)
   ▼
Render (如何展示？)
      ├── 补全菜单 (pum)
      ├── 文档窗口 (hover docs)
      ├── Ghost text (虚影文本)
      └── 签名帮助 (signature help)
```

---

## 2. 逐行注解：blink.cmp 完整配置

```lua
-- ============================================================================
-- lua/plugins/completion.lua — blink.cmp 完整补全配置
-- 文件：~/.config/nvim/lua/plugins/completion.lua
-- ============================================================================

return {
  {
    -- (1) 插件标识符
    "saghen/blink.cmp",

    -- (2) version = "1.*" — 使用 semver 范围锁定大版本
    --     1.* 表示接受 1.0.0 以上、2.0.0 以下的任何版本
    --     重要：blink.cmp 的 V2 分支有大量 breaking changes，
    --     使用 "1.*" 确保稳定，避免 V2 的破坏性变更
    version = "1.*",

    -- (3) dependencies — 运行时依赖
    dependencies = {
      -- (3a) friendly-snippets — 预置的代码片段集合
      --      包含 Lua、Python、JavaScript 等数十种语言的标准片段
      --      blink.cmp 的 snippets 源会自动发现并使用此包
      "rafamadriz/friendly-snippets",

      -- (3b) 可选：saghen/blink.compat — nvim-cmp 源的兼容层
      --      如果你需要尚未有 blink 原生替代的第三方 cmp 源
      --      { "saghen/blink.compat", optional = true, opts = {} },
    },

    -- (4) opts — 自动调用 require("blink.cmp").setup(opts)
    --     注意：blink.cmp 内部的 setup 函数会自动完成所有初始化
    --     包括注册 LSP capabilities、设置完成菜单、创建自动命令等
    opts = {

      -- ═══════════════════════════════════════════════════
      -- 4.1 keymap — 按键映射预设
      -- ═══════════════════════════════════════════════════
      keymap = {
        -- (4a) preset = "enter" — 使用 "enter" 预设方案
        --      四种预设：
        --        "default" — <C-y> 接受（Vim 传统风格）
        --        "super-tab" — <Tab> 接受/跳转（VS Code 风格）
        --        "enter" — <CR> 接受，<Tab> 仅跳转片段
        --        "none" — 完全不设置映射（完全手动控制）
        --      "enter" 是最推荐的方案，因为：
        --      - <CR>（回车）接受补全，直觉明确
        --      - <Tab> 专门用于 snippet 跳转，无歧义
        --      - 避免 <Tab> 在补全菜单和 snippet 之间的行为冲突
        preset = "enter",

        -- (4b) 自定义映射覆盖 preset 中的默认值
        --      格式：按键 = { 动作列表, 回退行为 }
        --      "select_and_accept" — 选中当前高亮的候选项并接受
        --      "fallback" — 如果 blink 不处理，传递给 Neovim 默认行为
        ["<C-y>"] = { "select_and_accept", "fallback" },
      },

      -- ═══════════════════════════════════════════════════
      -- 4.2 appearance — 外观配置
      -- ═══════════════════════════════════════════════════
      appearance = {
        -- (4c) nerd_font_variant = "mono" — Nerd Font 图标变体
        --      "mono" — 单色等宽图标（推荐，不影响字符对齐）
        --      "normal" — 标准 Nerd Font 图标
        --      "none" — 不使用图标（纯文本模式）
        nerd_font_variant = "mono",
      },

      -- ═══════════════════════════════════════════════════
      -- 4.3 completion — 补全行为
      -- ═══════════════════════════════════════════════════
      completion = {

        -- (4d) documentation — 自动显示文档窗口
        documentation = {
          -- (4e) auto_show = true — 选中候选项时自动弹出文档窗口
          auto_show = true,

          -- (4f) auto_show_delay_ms = 200 — 延迟 200ms 后弹出
          --      避免快速切换候选项时频繁弹出造成的闪烁
          auto_show_delay_ms = 200,

          -- (4f1) window.border — 文档窗口的边框样式
          --      会被主题的 BlinkCmpDocBorder 高亮组上色
          window = { border = "rounded" },
        },

        -- (4g) accept — 接受补全项时的行为
        accept = {
          -- (4h) auto_brackets — 自动添加括号
          --      基于 LSP 的 semantic tokens 判断是否需要括号
          --      例如：补全函数名 → 自动添加 ()
          --      实验性功能，需要 LSP 服务器支持 semantic tokens
          auto_brackets = { enabled = true },
        },

        -- (4i) ghost_text — 虚影文本（行内预显接受后的内容）
        ghost_text = {
          -- (4j) enabled = true — 启用虚影文本
          --      效果：光标后有灰色文本预显当前高亮候选项
          --      类似 GitHub Copilot 的 ghost text，但不依赖 AI
          enabled = true,
        },

        -- (4k) menu — 补全菜单行为
        menu = {
          -- (4l) draw — 菜单绘制配置
          draw = {
            -- (4m) columns — 菜单列定义
            --      第一列 (kind_icon + kind)：补全项的类型图标 + 类型名
            --      第二列 (label + label_description)：候选项标签 + 描述
            columns = { { "kind_icon" }, { "label", "label_description", gap = 1 } },
          },
        },

        -- (4n) trigger — 触发补全的条件
        trigger = {
          -- (4o) show_in_snippet = true — 是否在 snippet 内部也显示补全
          --      false 可避免 snippet 内弹出补全菜单干扰 Tab 跳转
          --      只在使用 super-tab 预设时相关
          show_in_snippet = true,
        },
      },

      -- ═══════════════════════════════════════════════════
      -- 4.4 sources — 补全数据源
      -- ═══════════════════════════════════════════════════
      sources = {

        -- (4p) default — 默认激活的源（在所有文件类型中）
        default = { "lsp", "path", "snippets", "buffer" },
        -- 优先级从左到右下降：
        --   "lsp" — LSP 服务器提供的语义补全（最高质量）
        --   "path" — 文件系统路径补全（输入 / 或 ./ 时触发）
        --   "snippets" — 代码片段（来自 friendly-snippets）
        --   "buffer" — 当前缓冲区的单词（Tab 补全的后备）

        -- (4q) per_filetype — 按文件类型定制的源列表
        --      键 = 文件类型
        --      值 = 源列表（继承默认 + 追加/移除）
        --      使用 inherit_defaults = true 在默认基础上加减
        per_filetype = {
          -- (4r) 对 SQL 文件添加 vim-dadbod 的补全源
          sql = { "dadbod" },

          -- (4s) 对 Lua 文件在默认源基础上追加 lazydev 源
          --      lazydev 提供 lazy.nvim 配置的补全
          lua = { inherit_defaults = true, "lazydev" },
        },

        -- (4t) providers — 自定义源提供者
        providers = {
          -- (4u) dadbod — 数据库补全源
          dadbod = {
            -- module — 实现此源的 Lua 模块路径
            module = "vim_dadbod_completion.blink",
          },

          -- (4v) lazydev — lazy.nvim 类型系统补全
          lazydev = {
            module = "lazydev.integrations.blink",
            -- score_offset — 分数偏移，调整候选项在列表中的位置
            --    正数提升排名，负数降低排名
            score_offset = 100,
          },
        },
      },

      -- ═══════════════════════════════════════════════════
      -- 4.5 signature — 签名帮助（函数参数提示）
      -- ═══════════════════════════════════════════════════
      signature = {
        -- (4w) enabled = true — 在输入函数参数时显示函数签名
        --      实验性功能！可能不稳定。
        --      需要 LSP 服务器支持 textDocument/signatureHelp
        --      Neovim 0.11 还内置了 <C-S> 快速触发签名帮助
        enabled = true,
      },

      -- ═══════════════════════════════════════════════════
      -- 4.6 fuzzy — 模糊匹配引擎
      -- ═══════════════════════════════════════════════════
      fuzzy = {
        -- (4x) implementation = "prefer_rust" — 优先使用 Rust 实现
        --      可选值：
        --        "prefer_rust" — Rust 可用时使用，不可用时降级到 Lua
        --        "prefer_rust_with_warning" — 同 prefer_rust，但 Rust 不可用时弹出警告
        --        "lua" — 强制使用纯 Lua 实现（无需编译，但速度较慢）
        --      Rust 实现约比 Lua 快 6 倍，且支持 SIMD 并行
        implementation = "prefer_rust",

        -- (4y) frecency — 启用 frecency 记分（综合考虑选择频率和最近使用时间）
        --      enabled = true 表示启用，经常选中的候选项会自动排到前面
        --      path 指定 frecency 持久化数据文件路径（使用默认值即可）
        frecency = { enabled = true },

        -- (4z) use_proximity = true — 启用 proximity 记分
        --      光标附近的单词获得加分，因为局部变量通常更相关
        use_proximity = true,

        -- (4aa) max_typos = 1 — 最大容错字符数
        --      允许在搜索时犯 1 个字符的拼写错误
        --      例如输入 "acmo" 可以匹配 "acme"
        --      设 0 禁用容错
        max_typos = 1,

        -- (4bb) sorts — 排序特征列表
        --      控制排序时考虑的维度优先级
        sorts = { "score", "exact", "kind" },
        --      "score" — 模糊匹配分数（主要排序依据）
        --      "exact" — 精确匹配优先
        --      "kind" — LSP CompletionItemKind 默认排序
      },
    },

    -- (5) opts_extend — 允许从其他 lazy.nvim spec 文件扩展此表
    --     例如在 lua/plugins/lang/python.lua 中通过 opts_extend 追加 python 特定的源
    --     未设置此字段的属性在合并时会被覆盖，设置了会被合并
    --     sources.default 是最常见的需要扩展的字段
    opts_extend = {
      -- (5a) 允许其他 spec 向 sources.default 数组追加新源
      --      例如：{ "saghen/blink.cmp", opts = { sources = { default = { "custom_source" } } } }
      --      这样的追加会合并而不是覆盖
      "sources.default",
    },
  },
}
```

---

## 3. 逐行注解：LSP Capabilities 集成

```lua
-- ============================================================================
-- lua/config/lsp.lua — blink.cmp 与 LSP 的桥接
-- 文件：~/.config/nvim/lua/config/lsp.lua
-- ============================================================================

-- (1) require("blink.cmp").get_lsp_capabilities() — 获取 blink.cmp 定义的 LSP 能力集
--     这个函数返回一个 table，告诉 LSP 服务器：「Neovim 客户端支持以下功能」
--     如果不传 blink 的 capabilities，LSP 服务器可能不返回某些补全信息
local capabilities = require("blink.cmp").get_lsp_capabilities({
  -- (2) 第一个参数：额外的 capabilities 覆盖
  --     snippetSupport = true — 告诉服务器客户端支持 snippet 格式的补全项
  --     大部分现代 LSP 服务器依赖此标志来决定是否返回带 tabstop 的补全
  textDocument = {
    completion = {
      completionItem = {
        snippetSupport = true,  -- 关键！否则函数片段无参数占位符
      },
    },
  },
}, -- (3) 第二个参数：是否合并默认 capabilities（默认 true）
   --     设为 false 只返回你指定的 capabilities，不合并 blink 的默认值
   true)

-- (4) 将 capabilities 传给所有 LSP 服务器的默认配置
--     使用 "*" 通配符表示「所有服务器」
--     这样无需在每个服务器的 vim.lsp.config() 中重复设置
vim.lsp.config("*", {
  capabilities = capabilities,
  -- 所有 LSP 服务器都会继承此配置
})
```

---

## 4. 从 nvim-cmp 迁移对照表

| nvim-cmp 配置 | blink.cmp 等价 | 说明 |
|--------------|---------------|------|
| `sources = { { name = "nvim_lsp" } }` | `sources.default = { "lsp" }` | 内置，无需插件 |
| `sources = { { name = "buffer" } }` | `sources.default = { "buffer" }` | 内置 |
| `sources = { { name = "luasnip" } }` | `sources.default = { "snippets" }` | 内置，默认用 vim.snippet |
| `mapping = cmp.mapping.preset.insert()` | `keymap.preset = "enter"` | 一行替代 20+ 行 |
| `experimental.ghost_text = true` | `completion.ghost_text.enabled = true` | 内置 |
| `window.documentation = { ... }` | `completion.documentation.auto_show = true` | 内置 |
| `cmp.mapping.confirm({ select = true })` | `preset = "enter"` | `<CR>` 接受 |
| `sorting = { comparators = {...} }` | `fuzzy.sorts = { ... }` | 简化 |
| `snippet.expand = function(args) ... end` | 无需配置 | 内置 vim.snippet |

---

## 5. 片段引擎说明

blink.cmp 默认使用 **Neovim 0.10+ 的内置 `vim.snippet`**。你不需要安装 LuaSnip 或配置任何片段引擎。

```lua
-- 无需任何配置！blink.cmp 自动处理：
--   1. 从 friendly-snippets 加载 VSCode 格式的片段
--   2. 从 ~/.config/nvim/snippets/ 加载自定义片段
--   3. 在补全菜单中混合显示 LSP 结果和片段
```

**如果必须使用 LuaSnip**（需要 choiceNode/dynamicNode 高级功能）：

```lua
snippets = {
  preset = "luasnip",  -- 从默认的 vim.snippet 切换到 LuaSnip
}
```

---

## 6. 关键参考

| 资源 | URL |
|------|-----|
| blink.cmp 官方文档 | https://cmp.saghen.dev/ |
| blink.cmp GitHub | https://github.com/saghen/blink.cmp |
| blink.cmp vs nvim-cmp (作者对比) | https://gist.github.com/saghen/e731f6f6e30a4c01f6bc7cdaa389d463 |
| LazyVim Blink Extra | https://lazyvim.github.io/extras/coding/blink.html |
| friendly-snippets | https://github.com/rafamadriz/friendly-snippets |
