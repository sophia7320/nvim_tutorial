# which-key.nvim v3 — 按键提示

> **TL;DR**: which-key v3（2024 年重写）使用全新的 `wk.add()` API。按 `<leader>` 后弹出所有可用快捷键的菜单。**必须为所有 keymap 添加 `desc` 才能生效。**

---

## 1. v3 基础配置

```lua
-- lua/plugins/editor.lua
{
  "folke/which-key.nvim",
  event = "VeryLazy",
  opts = {
    -- 预设主题（classic | modern | helix）
    preset = "modern",
    -- 延迟（ms），独立于 timeoutlen
    delay = 300,
    -- 触发器
    triggers = {
      { "<auto>", mode = "nxsot" },
      { "s", mode = "v" },
    },
    -- 插件配置
    plugins = {
      marks = true,     -- 显示标记
      registers = true, -- 显示寄存器
      spelling = { enabled = true, suggestions = 20 },
      presets = {
        operators = true,  -- d, y, c 等操作符提示
        motions = true,    -- 移动提示
        text_objects = true,
        windows = true,
        nav = true,
        z = true,
        g = true,
      },
    },
    -- 手动注册的按键组（可选，blink.cmp / keymaps.lua 中的 desc 会自动发现）
    spec = {
      { "<leader>f", group = "Find / 查找" },
      { "<leader>g", group = "Git" },
      { "<leader>l", group = "LSP" },
      { "<leader>c", group = "Code / 代码" },
      { "<leader>d", group = "Debug / 调试" },
      { "<leader>t", group = "Toggle / Terminal" },
      { "<leader>u", group = "UI / 界面" },
      { "<leader>s", group = "Search / 搜索" },
    },
  },
}
```

> 来源：[which-key.nvim README (v3)](https://github.com/folke/which-key.nvim)

---

## 2. 已定义的 Keymaps 自动发现

`which-key` v3 **自动扫描**你在 `keymaps.lua` 和 lazy.nvim `keys` 中定义的所有映射。只要使用了 `desc` 参数：

```lua
-- 自动被 which-key 发现并展示！
vim.keymap.set("n", "<leader>ff", "<cmd>Telescope find_files<CR>",
  { desc = "查找文件" })
vim.keymap.set("n", "<leader>fg", "<cmd>Telescope live_grep<CR>",
  { desc = "实时搜索" })
```

按下 `<leader>f` 后自动显示：
```
f → +Find
  f → 查找文件
  g → 实时搜索
  b → 缓冲区列表
  h → 帮助文档
  o → 最近文件
```

---

## 3. v2 → v3 迁移

v3 完全重写了 API：

```lua
-- ❌ v2 风格（已废弃）
require("which-key").register({
  f = {
    name = "Find",
    f = { "<cmd>Telescope find_files<CR>", "查找文件" },
  },
})

-- ✅ v3 风格：使用 spec 字段
{
  "folke/which-key.nvim",
  opts = {
    spec = {
      { "<leader>ff", "<cmd>Telescope find_files<CR>", desc = "查找文件" },
    },
  },
}

-- ✅ v3 风格：使用 wk.add()
local wk = require("which-key")
wk.add({
  { "<leader>ff", "<cmd>Telescope find_files<CR>", desc = "查找文件" },
})
```

---

## 4. 高级：动态扩展缓冲区 Keymaps

```lua
-- 自动展开缓冲区本地 keymaps
{ "<leader>b", group = "Buffers", expand = function()
    return require("which-key.extras").expand.buf()
  end,
}
```

---

## 5. 常用命令

| 命令 | 说明 |
|------|------|
| `:WhichKey <leader>` | 显示指定前缀的快捷键 |
| `:WhichKey ""` | 显示所有快捷键 |
| `:WhichKey "g"` | 显示 g 开头的快捷键 |

---

## 6. 关键参考

| 资源 | URL |
|------|-----|
| which-key.nvim v3 README | https://github.com/folke/which-key.nvim |
| v3 更新说明 | https://github.com/folke/which-key.nvim/blob/main/NEWS.md |
