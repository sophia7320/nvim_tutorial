# AI 编码助手 — Neovim 中的 AI 集成

> **TL;DR**: 推荐组合：**copilot.lua**（内联补全）+ **CodeCompanion.nvim**（AI 聊天）。CodeCompanion 是 2025-2026 年最推荐的多后端 AI 平台。

---

## 1. 各插件定位对比

| 插件 | 定位 | 后端 | 推荐场景 |
|------|------|------|----------|
| **copilot.lua** | 内联代码补全 | GitHub Copilot | Tab 补全（首选） |
| **CodeCompanion.nvim** | 通用 AI 平台 | Claude/OpenAI/Copilot/Ollama | **推荐** 聊天 + 内联编辑 |
| **CopilotChat.nvim** | Copilot 聊天 | GitHub Copilot | Copilot 订阅用户 |
| **avante.nvim** | Cursor 风格 | Claude/OpenAI | 重构/审查（diff-first） |

> 对比来源：[avante vs CodeCompanion 2026](https://samuellawrentz.com/blog/neovim-ai-plugins-avante-codecompanion/) | [CodeCompanion GitHub](https://github.com/olimorris/codecompanion.nvim)

---

## 2. 推荐组合：copilot.lua + CodeCompanion

### 2.1 copilot.lua（内联补全）

```lua
-- lua/plugins/ai.lua
return {
  -- GitHub Copilot 内联补全
  {
    "zbirenbaum/copilot.lua",
    cmd = "Copilot",
    event = "InsertEnter",
    opts = {
      suggestion = {
        enabled = true,
        auto_trigger = false, -- 手动触发，减少干扰
        keymap = {
          accept = "<M-l>",   -- Alt+l 接受建议
          next = "<M-]>",     -- Alt+] 下一个建议
          prev = "<M-[>",     -- Alt+[ 上一个建议
          dismiss = "<C-]>",  -- Ctrl+] 关闭
        },
      },
      panel = { enabled = true },
    },
  },
}
```

### 2.2 CodeCompanion.nvim（AI 聊天）

```lua
  -- CodeCompanion：统一 AI 接口
  {
    "olimorris/codecompanion.nvim",
    dependencies = {
      "nvim-lua/plenary.nvim",
      "nvim-treesitter/nvim-treesitter",
    },
    keys = {
      -- 内联助手（在光标位置插入 AI 代码）
      { "<leader>aa", "<cmd>CodeCompanion<CR>", mode = { "n", "v" }, desc = "AI 内联助手" },
      -- 聊天缓冲区
      { "<leader>ac", "<cmd>CodeCompanionChat<CR>", desc = "AI 聊天" },
      -- 命令生成
      { "<leader>aC", "<cmd>CodeCompanionCmd<CR>", desc = "AI 命令" },
    },
    opts = {
      -- 策略配置：在这里选择使用的后端
      strategies = {
        chat = { adapter = "copilot" },     -- 使用 Copilot（免费/订阅）
        inline = { adapter = "copilot" },
        cmd = { adapter = "copilot" },
      },
      -- 自定义适配器（示例：添加 Ollama 本地模型）
      -- adapters = {
      --   ollama = function()
      --     return require("codecompanion.adapters").extend("ollama", {
      --       name = "ollama",
      --     })
      --   end,
      -- },
    },
    config = function(_, opts)
      require("codecompanion").setup(opts)
    end,
  },
```

**支持的适配器（开箱即用）**：
- `copilot` — GitHub Copilot（免费/订阅）
- `anthropic` — Claude（API 按量付费）
- `openai` — GPT-4o（API 按量付费）
- `gemini` — Google Gemini
- `ollama` — 本地模型（隐私/免费）
- `deepseek` — DeepSeek
- `azure_openai` — Azure OpenAI

**CodeCompanion 的核心功能**：

| 命令 | 功能 |
|------|------|
| `:CodeCompanionChat` | 打开 AI 聊天缓冲区 |
| `:CodeCompanion <prompt>` | 内联编辑（在光标位置插入 AI 代码） |
| `:CodeCompanionCmd` | 生成 Vim 命令 |
| `:CodeCompanionActions` | 打开操作面板 |

**编辑器上下文变量**：
- `@lsp` — 注入当前文件的 LSP 诊断
- `@buffers` — 注入所有打开缓冲区
- `@editor` — 注入当前文件的代码
- `#buffer` — 引用特定缓冲区

> 来源：[CodeCompanion 官方文档](https://codecompanion.olimorris.dev/) | [CodeCompanion GitHub](https://github.com/olimorris/codecompanion.nvim)

---

## 3. 备选：avante.nvim（Cursor 风格）

如果你偏好 Cursor IDE 的 diff-first 工作流：

```lua
{
  "yetone/avante.nvim",
  build = "make",
  opts = {
    provider = "claude",
    providers = {
      claude = {
        model = "claude-sonnet-4-20250514",
        timeout = 30000,
      },
    },
  },
  keys = {
    { "<leader>av", "<cmd>AvanteToggle<CR>", desc = "Avante AI" },
  },
}
```

**avante vs CodeCompanion 选型建议**：
- **重构/审查** → avante（diff-first 工作流）
- **日常问答/调试** → CodeCompanion（轻量、灵活）
- **两者都用** → 绑定不同快捷键，互补不冲突

---

## 4. 系统依赖

```bash
# CodeCompanion / copilot.lua
# 无额外依赖，通过 API 通信

# 本地模型
curl -fsSL https://ollama.com/install.sh | sh  # Ollama
ollama pull codellama:7b                          # 下载模型
```

---

## 5. 按键最佳实践

```lua
-- 内联补全
vim.keymap.set("i", "<M-l>", function() require("copilot.suggestion").accept() end,
  { desc = "接受 Copilot 建议" })

-- AI 聊天
vim.keymap.set({ "n", "v" }, "<leader>aa", "<cmd>CodeCompanion<CR>",
  { desc = "AI 内联" })
vim.keymap.set("n", "<leader>ac", "<cmd>CodeCompanionChat<CR>",
  { desc = "AI 聊天" })
```

---

## 6. 关键参考

| 资源 | URL |
|------|-----|
| copilot.lua | https://github.com/zbirenbaum/copilot.lua |
| CodeCompanion.nvim | https://github.com/olimorris/codecompanion.nvim |
| CodeCompanion 文档 | https://codecompanion.olimorris.dev/ |
| avante.nvim | https://github.com/yetone/avante.nvim |
| avante vs CodeCompanion 2026 | https://samuellawrentz.com/blog/neovim-ai-plugins-avante-codecompanion/ |
| CopilotChat.nvim | https://github.com/CopilotC-Nvim/CopilotChat.nvim |
