# blink.cmp — 配置键速查

> **TL;DR**: 默认值已足够好。只改你有明确需求的。详细教程见 `04-completion/blink-cmp.md`。

## 顶层字段

| 字段 | 类型 | 默认 | 说明 |
|------|------|------|------|
| `version` | `string` | — | 🟢 `"1.*"` 锁定大版本 |
| `dependencies` | `string[]` | `{}` | 🟢 `"rafamadriz/friendly-snippets"` |
| `opts_extend` | `string[]` | — | 🟢 `{ "sources.default" }` |

## keymap

| key | 类型 | 默认 | 说明 |
|-----|------|------|------|
| `keymap.preset` | `string` | `"default"` | `"default"` / `"enter"` / `"super-tab"` / `"none"` |
| `keymap["<C-y>"]` | `table` | — | 自定义：`{ "select_and_accept", "fallback" }` |

| preset | 接受键 | 特点 |
|--------|--------|------|
| `"default"` | `<C-y>` | Vim 传统风格 |
| `"super-tab"` | `<Tab>` | VS Code 迁移 |
| **`"enter"`** | `<CR>` | **推荐**<br>Enter 接受，Tab 专用于片段跳转 |

## appearance

| key | 类型 | 默认 | 说明 |
|-----|------|------|------|
| `appearance.nerd_font_variant` | `string` | `"none"` | `"mono"` / `"normal"` |

## completion

| key | 类型 | 默认 | 说明 |
|-----|------|------|------|
| `completion.documentation.auto_show` | `boolean` | `false` | 自动弹出文档 |
| `completion.documentation.auto_show_delay_ms` | `number` | `0` | 延迟弹出 |
| `completion.documentation.window.border` | `string` | — | `"rounded"` / `"single"` |
| `completion.accept.auto_brackets.enabled` | `boolean` | `false` | 函数补全自动加 `()` |
| `completion.ghost_text.enabled` | `boolean` | `false` | 幽灵文本预览 |
| `completion.menu.draw.columns` | `table` | `{ {"kind"}, {"label", "desc"} }` | 菜单列定义 |
| `completion.trigger.show_in_snippet` | `boolean` | — | 片段内是否显示补全 |

## sources

| key | 类型 | 默认 | 说明 |
|-----|------|------|------|
| `sources.default` | `string[]` | `{"lsp","path","snippets","buffer"}` | 全局默认源 |
| `sources.per_filetype` | `table` | `{}` | `sql = { "dadbod" }` |
| `sources.providers` | `table` | `{}` | 自定义源注册 |

**内置源优先级**: `lsp` > `path` > `snippets` > `buffer`（左高右低）

## fuzzy

| key | 类型 | 默认 | 说明 |
|-----|------|------|------|
| `fuzzy.implementation` | `string` | `"lua"` | `"prefer_rust"` / `"lua"` |
| `fuzzy.frecency.enabled` | `boolean` | `true` | 频率+最近使用排序 |
| `fuzzy.frecency.path` | `string` | `stdpath("state")/blink/cmp/frecency.dat` | 持久化路径 |
| `fuzzy.use_proximity` | `boolean` | `true` | 光标附近加分 |
| `fuzzy.max_typos` | `number` | `0` | 容错字符数，`1` = 允许 1 个拼写错误 |
| `fuzzy.sorts` | `string[]` | `{"score","exact","kind"}` | 排序维度优先级 |

## signature

| key | 类型 | 默认 | 说明 |
|-----|------|------|------|
| `signature.enabled` | `boolean` | `false` | 实验性，函数参数签名提示 |

## LSP Capabilities

```lua
local capabilities = require("blink.cmp").get_lsp_capabilities({
  textDocument = { completion = {
    completionItem = { snippetSupport = true },
  }},
}, true)
vim.lsp.config("*", { capabilities = capabilities })
```
