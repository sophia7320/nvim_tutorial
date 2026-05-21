# GitHub Copilot + CodeCompanion — AI 编码助手深度解析

> **TL;DR**: 推荐双轨制：**copilot.lua**（内联 Tab 补全）+ **CodeCompanion.nvim**（AI 聊天）。本章逐句解析两者完整配置。

---

## 1. 生态定位速查

| 插件 | 定位 | 后端 | 何时用 |
|------|------|------|--------|
| **copilot.lua** | 内联代码补全 | GitHub Copilot | Tab 补全 |
| **CodeCompanion** | 通用 AI 平台 | Claude/OpenAI/Copilot/Ollama | 聊天/重构/解答 |

---

## 2. 逐行注解：copilot.lua

```lua
-- ============================================================================
-- lua/plugins/ai.lua — GitHub Copilot 内联补全
-- ============================================================================

return {
  {
    -- (1) 纯 Lua Copilot 客户端，替代官方 copilot.vim（Node.js 版）
    "zbirenbaum/copilot.lua",

    -- (2) cmd + event 驱动的懒加载
    --     ╔══════════════════════════════════════════╗
    --     ║ cmd/event 语义详见：                       ║
    --     ║ → 02-plugin-manager/lazy-nvim.md §3.2    ║
    --     ╚══════════════════════════════════════════╝
    cmd = "Copilot",
    event = "InsertEnter",

    opts = {
      suggestion = {
        enabled = true,

        -- (3) auto_trigger = false — 禁用自动触发
        --     按需手动请求建议，避免与 blink.cmp 的补全菜单冲突
        auto_trigger = false,

        -- (4) keymap — 用 <M-> (Meta/Alt) 前缀避免冲突
        keymap = {
          accept = "<M-l>",    -- (4a) Alt+l → 接受当前建议
          next = "<M-]>",      -- (4b) Alt+] → 下一个备选方案
          prev = "<M-[>",      -- (4c) Alt+[ → 上一个备选方案
          dismiss = "<C-]>",   -- (4d) Ctrl+] → 关闭建议
        },
      },
      panel = {
        enabled = true,         -- (5) :Copilot panel 侧边栏浏览多个建议
      },
      -- (6) 指定 Node.js 路径（>= v22）
      copilot_node_command = "node",
    },
  },
}
```

---

## 3. 逐行注解：CodeCompanion.nvim

```lua
-- ============================================================================
-- lua/plugins/ai.lua 续 — CodeCompanion AI 聊天
-- ============================================================================

return {
  {
    -- (7) 多后端 AI 接口 — 一个插件管理所有 LLM
    "olimorris/codecompanion.nvim",

    -- (8) plenary 提供异步 HTTP/JSON 工具
    --     treesitter 为聊天 buffer 提供语法高亮
    dependencies = {
      "nvim-lua/plenary.nvim",
      "nvim-treesitter/nvim-treesitter",
    },

    -- (9) 快捷键：keys 驱动的懒加载
    keys = {
      -- (9a) <leader>aa → 内联助手（在光标处插入 AI 代码）
      --      在 Visual 模式则会基于选中内容
      { "<leader>aa", "<cmd>CodeCompanion<CR>",
        mode = { "n", "v" }, desc = "AI 内联" },
      -- (9b) <leader>ac → 聊天缓冲区（Ctrl+S 发送消息）
      { "<leader>ac", "<cmd>CodeCompanionChat<CR>",
        desc = "AI 聊天" },
      -- (9c) <leader>aC → 自然语言生成 Vim 命令
      { "<leader>aC", "<cmd>CodeCompanionCmd<CR>",
        desc = "AI 命令" },
    },

    opts = {
      -- (10) strategies — 不同交互模式可指定不同后端
      strategies = {
        -- (10a) 聊天用 Copilot（订阅用户无需额外 API key）
        chat = { adapter = "copilot" },
        inline = { adapter = "copilot" },
        cmd = { adapter = "copilot" },

        -- (10b) 没有 Copilot？换用其他后端：
        -- chat = { adapter = "anthropic" },  -- 需要 ANTHROPIC_API_KEY
        -- chat = { adapter = "openai" },     -- 需要 OPENAI_API_KEY
        -- chat = { adapter = "ollama" },     -- 本地免费模型
      },

      -- (11) 自定义适配器——添加 Ollama 本地模型
      -- adapters = {
      --   ollama_local = function()
      --     return require("codecompanion.adapters").extend("ollama", {
      --       -- 所有参数均已有合理默认值，只需提供名称
      --     })
      --   end,
      -- },
    },
  },
}
```

---

## 4. CodeCompanion 适配器速查

| 适配器 | 后端 | 费用模式 | 环境变量 |
|--------|------|----------|----------|
| `copilot` | GitHub Copilot | 订阅 / 免费额度 | 无（通过 copilot.lua 认证） |
| `anthropic` | Claude | API 按量 | `ANTHROPIC_API_KEY` |
| `openai` | GPT-4o | API 按量 | `OPENAI_API_KEY` |
| `gemini` | Google Gemini | API 按量 | `GEMINI_API_KEY` |
| `ollama` | 本地模型 | **免费** | 无（需先 `ollama pull <model>`） |

---

## 5. 对比：三大 AI 插件

| 功能 | copilot.lua | CodeCompanion | avante.nvim |
|------|:-----------:|:-------------:|:-----------:|
| Tab 补全 | ✅ **核心** | ❌ | ❌ |
| AI 聊天 | ❌ | ✅ | ✅ |
| 多后端 | ❌ | ✅ 7+ | ✅ 4+ |
| LSP 诊断注入 | ❌ | ✅ `@lsp` | 部分 |
| 内联编辑 | ❌ | ✅ | ✅ diff-first |

**推荐**：copilot.lua + CodeCompanion 互补。一个补全，一个聊天。绑定不同快捷键，无冲突。

---

## 6. 关键参考

| 资源 | URL |
|------|-----|
| copilot.lua | https://github.com/zbirenbaum/copilot.lua |
| CodeCompanion.nvim | https://codecompanion.olimorris.dev/ |
| avante.nvim | https://github.com/yetone/avante.nvim |
