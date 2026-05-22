# lazy.nvim — Plugin Spec 全字段速查

> **TL;DR**: 已熟悉 lazy.nvim 的读者快速查阅 Spec 字段。详细教程见 `02-plugin-manager/lazy-nvim.md`。

## Spec 来源

| 字段 | 类型 | 示例 |
|------|------|------|
| 短 URL | `string` | `"folke/tokyonight.nvim"` |
| 完整 URL | `string` | `"https://github.com/folke/tokyonight.nvim"` |
| 本地目录 | `table` | `{ dir = "~/projects/my-plugin" }` |

## 加载控制

| 字段 | 类型 | 默认 | 说明 |
|------|------|------|------|
| `lazy` | `boolean` | `true` | `false` 立即加载（配色必设） |
| `event` | `string\|string[]` | — | 事件触发：`"VeryLazy"` / `"BufReadPre"` |
| `cmd` | `string\|string[]` | — | 命令触发：`"Telescope"` |
| `keys` | `table[]` | — | 按键触发：`{ "<leader>e", ... }` |
| `ft` | `string\|string[]` | — | 文件类型触发：`"norg"` |
| `cond` | `function` | — | 条件加载：`not vim.g.vscode` |
| `priority` | `number` | `50` | 加载优先级；配色方案设 `1000` |

## 配置方式

| 字段 | 类型 | 说明 |
|------|------|------|
| `opts` | `table` | **推荐**：自动调用 `require("plugin").setup(opts)` |
| `config` | `function` | 需自定义逻辑时使用（不是只调 `setup` 时用 `opts`） |
| `init` | `function` | 插件加载前执行（设置 vim.g 全局变量） |

## 版本控制

| 字段 | 类型 | 说明 |
|------|------|------|
| `branch` | `string` | `"main"` |
| `tag` | `string` | `"0.1.8"` |
| `version` | `string` | semver：`"1.*"` |
| `commit` | `string` | ⚠️ 不推荐（维护麻烦） |
| `pin` | `boolean` | `true` 锁定不更新 |

## 依赖

| 字段 | 类型 | 说明 |
|------|------|------|
| `dependencies` | `string[]\|Spec[]` | 自动跟随加载 |
| `build` | `string\|function` | 安装后编译：`build = ":TSUpdate"` / `build = "make"` |

## 其他

| 字段 | 类型 | 默认 | 说明 |
|------|------|------|------|
| `name` | `string` | 自动推断 | 短名：`"catppuccin"` |
| `enabled` | `boolean` | `true` | 多主题共存的开关 |

## 常用命令

| 命令 | 说明 |
|------|------|
| `:Lazy` | 管理面板 |
| `:Lazy sync` | 安装/更新/删除 |
| `:Lazy update` | 更新全部 |
| `:Lazy clean` | 删除不在配置中的插件 |
| `:Lazy restore` | 按 `lazy-lock.json` 恢复 |
| `:Lazy profile` | 启动性能分析 |
