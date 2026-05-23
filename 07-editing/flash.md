# flash.nvim — 快速跳转

> **TL;DR**: flash.nvim 是 Leap/Hop 的现代替代品。`s` 键跳转、`S` 键 Treesitter 跳转、`r` 键远程操作。集成搜索 `/` `?` 实现标签化跳转。

---

## 1. 基础配置

```lua
-- lua/plugins/editor.lua
{
  "folke/flash.nvim",
  event = "VeryLazy",
  opts = {
    -- 搜索模式配置
    search = {
      multi_window = true, -- 跨窗口搜索 [默认值，可选]
    },
    -- 标签样式
    labels = "asdfghjklqwertyuiopzxcvbnm", -- [默认值，可选]
    -- 高亮组
    highlight = {
      backdrop = false, -- 不显示背景模糊 [默认值，可选]
    },
  },
  keys = {
    -- s: 普通跳转（normal + visual + operator-pending）
    {
      "s",
      mode = { "n", "x", "o" },
      function() require("flash").jump() end,
      desc = "Flash 跳转",
    },
    -- S: Treesitter 跳转
    {
      "S",
      mode = { "n", "x", "o" },
      function() require("flash").treesitter() end,
      desc = "Flash Treesitter 跳转",
    },
    -- r: 远程操作（operator-pending 模式）
    {
      "r",
      mode = "o",
      function() require("flash").remote() end,
      desc = "Flash 远程操作",
    },
    -- R: Treesitter 远程操作
    {
      "R",
      mode = { "o", "x" },
      function() require("flash").treesitter_search() end,
      desc = "Flash Treesitter 搜索",
    },
    -- 增强 / 和 ? 搜索
    {
      "<c-s>",
      mode = { "c" },
      function() require("flash").toggle() end,
      desc = "切换 Flash 搜索高亮",
    },
  },
}
```

> 来源：[flash.nvim README](https://github.com/folke/flash.nvim)

---

## 2. 核心模式

### 2.1 Jump Mode（`s`）

按下 `s`，然后输入目标位置的 2-3 个字符。flash 会自动给所有匹配位置打上标签（a, s, d, f, ...），再按一个标签键即可跳转。

```
s + "fu" → 所有 "fu" 出现位置打标签 → 按 "d" 跳转到第 4 个
```

### 2.2 Treesitter Mode（`S`）

按下 `S`，所有语法节点（函数、类、循环等）自动打标签，直接按键跳转。

```
S → 所有函数被标记 → 按 "f" 跳转到第 6 个函数
```

### 2.3 Remote Mode（`r`）

在 operator-pending 模式中（如 `d` 删除、`y` 复制等），`r` 让你**远程执行操作**：

```
d + r → 输入搜索字符 → 选中范围 → 删除
```

等价于"在远处执行删除"。

### 2.4 Search Integration（`<C-s>`）

在 `/` 或 `?` 搜索模式下，`<C-s>` 切换 flash 标签显示。搜索结果自动打标签，直接按键跳转。

---

## 3. 自定义标签

```lua
opts = {
  labels = "asdfghjklqwertyuiopzxcvbnm",     -- 默认
  -- labels = "jfkdls;ahgmbwrcnvz,x.uypqt",  -- Dvorak 布局
  -- labels = "1234567890",                  -- 数字标签
}
```

标签顺序即按键排布——越靠前的标签代表越近的匹配。

---

## 4. 与哪些按键冲突？

flash.nvim 占用了 `s` 键（默认的 substitute）。

**如果你需要保留 `s`（替换字符）**，可以使用其他快捷键：

```lua
keys = {
  { "<leader>s", mode = { "n", "x", "o" }, function() require("flash").jump() end, desc = "Flash" },
}
```

---

## 5. 关键参考

| 资源 | URL |
|------|-----|
| flash.nvim README | https://github.com/folke/flash.nvim |
| Operating Modes 详解 | https://deepwiki.com/folke/flash.nvim/4-operating-modes |
