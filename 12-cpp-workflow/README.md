# C++ / CMake / Git 全流程开发环境

> **TL;DR**: 以 C++ 开发者的视角，从零搭建 Neovim 下的完整工具链——LSP 补全、CMake 构建、DAP 调试、Git 版本控制。全程 step-by-step，每个步骤链接回前序教程的详细章节。

---

## 1. 前置阅读

> ⚠️ 本指南假设你已完成以下基础配置。若尚未完成，请先阅读对应章节再返回。

| 步骤 | 对应教程 | 你需要已配置的内容 |
|------|----------|-------------------|
| 插件管理器 | [02-plugin-manager/lazy-nvim.md](../02-plugin-manager/lazy-nvim.md) | lazy.nvim 已安装并能正常加载插件 |
| LSP 基础 | [03-lsp/overview.md](../03-lsp/overview.md) | mason.nvim 已就绪；需了解 `vim.lsp.config()` 详见 [03-lsp/lspconfig.md](../03-lsp/lspconfig.md) |
| 补全系统 | [04-completion/blink-cmp.md](../04-completion/blink-cmp.md) | blink.cmp 已工作 |
| DAP 基础 | [08-dap/dap.md](../08-dap/dap.md) | nvim-dap + nvim-dap-ui 已安装 |
| Git 集成 | [09-git/gitsigns.md](../09-git/gitsigns.md) | gitsigns + neogit 已配置 |

---

## 2. 系统依赖

```bash
# Ubuntu/Debian
sudo apt install clangd         # C/C++ LSP 服务器
sudo apt install clang-format   # 代码格式化
sudo apt install clang-tidy     # 静态分析/代码检查
sudo apt install codelldb       # 调试适配器（或通过 Mason 安装）
sudo apt install cmake          # 构建系统
sudo apt install bear           # 为非 CMake 项目生成 compile_commands.json

# CMake LSP（编辑 CMakeLists.txt 时提供补全和格式化）
# neocmakelsp 是 2025+ 推荐的新方案（Rust + Tree-sitter，替代已废弃的 cmake-language-server）
# 安装方式见步骤 6.3

# 验证安装
clangd --version                # >= 16 推荐
cmake --version                 # >= 3.16
```

---

## 3. 理解 compile_commands.json

### 3.1 它是什么？

`compile_commands.json` 是一份 **JSON 编译数据库**。它记录了项目中每个 `.cpp` 文件是如何编译的——包括编译器路径、include 路径、宏定义、编译选项。

```jsonc
// 一条典型记录
{
  "directory": "/home/user/myproject/build",   // 编译时的工作目录
  "command": "/usr/bin/c++ -Iinclude -std=c++20 -c src/main.cpp", // 完整的编译命令
  "file": "/home/user/myproject/src/main.cpp"   // 源文件路径
}
```

### 3.2 为什么 clangd 需要它？

**clangd 本身不是一个编译器**——但你写的代码可能包含 `-DDEBUG` 宏、`-I` include 路径、`-std=c++20` 等编译选项。没有这些信息，clangd 不知道：

- `#include "config.h"` 该去哪里找
- `#ifdef DEBUG` 应该走哪个分支
- C++20 的 `std::format` 是否可用

`compile_commands.json` **告诉 clangd 你的代码是如何编译的**，让它能精确地理解你的代码结构。

> 来源：[clangd 官方文档 - compile_commands](https://clangd.llvm.org/design/compile-commands)

---

## 4. 配置 CMake 项目生成 compile_commands.json

### 4.1 最小 CMakeLists.txt

```cmake
# (1) cmake_minimum_required — 指定 CMake 最低版本
#     3.16 是因为 CMAKE_EXPORT_COMPILE_COMMANDS 在此版本后默认支持
cmake_minimum_required(VERSION 3.16)

# (2) project — 声明项目名和语言
#     如果只用 C，写 project(myapp C)
project(myapp LANGUAGES CXX)

# (3) CMAKE_EXPORT_COMPILE_COMMANDS — 🔑 核心设置！
#     设为 ON 后，CMake 会在 build/ 目录下生成 compile_commands.json
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# (4) 设置 C++ 标准为 C++20（与 clangd 的 --std=c++20 一致）
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# (5) 添加可执行文件目标
add_executable(myapp
  src/main.cpp
  src/utils.cpp
)

# (6) 添加 include 目录（让编译器知道头文件在哪）
target_include_directories(myapp PRIVATE include)

# (7) 调试模式开启符号（给 gdb/lldb 用的 -g 标志）
set(CMAKE_BUILD_TYPE Debug)     # Debug: -g -O0; Release: -O3; RelWithDebInfo: -g -O2
```

### 4.2 配置并构建

```bash
# 在项目根目录执行
cmake -B build -DCMAKE_BUILD_TYPE=Debug   # -B build = 指定构建目录
cmake --build build -j$(nproc)            # --build = 执行编译; -j = 并行任务数

# 🎯 关键一步：将 compile_commands.json 链接到项目根目录
# clangd 默认从文件所在目录向上查找此文件
ln -sf build/compile_commands.json .
```

执行后你的目录结构：
```
myproject/
├── CMakeLists.txt
├── compile_commands.json  → build/compile_commands.json  (符号链接)
├── build/
│   ├── compile_commands.json   (CMake 自动生成的)
│   └── myapp                   (编译出的可执行文件)
├── include/
│   └── utils.h
└── src/
    ├── main.cpp
    └── utils.cpp
```

---

## 5. 配置 clangd LSP

### 5.1 安装与基本配置

clangd 通过 Mason 安装（已在 [03-lsp/overview.md](../03-lsp/overview.md) 中配置 Mason）：

```lua
-- lua/plugins/lsp.lua 中追加
{
  "mason-org/mason-lspconfig.nvim",
  opts = {
    ensure_installed = { "clangd" },  -- (1) 追加到已有列表中
    -- clangd 通过 Mason 自动安装，PATH 自动配置，无需手动干预
  },
}
```

### 5.2 clangd 自定义参数（逐行注解）

```lua
-- lua/config/lsp.lua 中追加

vim.lsp.config("clangd", {
  -- (2) cmd — clangd 的启动参数
  --     所有这些参数都是可选的；默认 `cmd = { "clangd" }` 已足够好
  cmd = {
    "clangd",

    -- (3) --background-index — 后台构建全项目索引
    --     首次打开项目时会消耗一些 CPU，但之后跳转定义/引用近乎即时
    "--background-index",

    -- (4) --clang-tidy — 启用 clang-tidy 静态分析
    --     在编辑器中实时显示 clang-tidy 的诊断（如性能建议、现代化提示）
    --     需要项目有 .clang-tidy 配置文件（见步骤 5）
    "--clang-tidy",

    -- (5) --header-insertion=never — 禁用自动添加 #include
    --     iwyu = "include what you use"（自动补全 include）
    --     never = 完全禁用（推荐：include 顺序在 C++ 中很重要，让开发者手动控制）
    "--header-insertion=never",

    -- (6) --completion-style=detailed — 补全菜单显示完整签名
    --     例如：func(int x, float y) -> int  而不是只显示 func
    "--completion-style=detailed",

    -- (7) --function-arg-placeholders — 函数补全后显示参数占位符
    --     补全 func( 后显示 func(${1:int x}, ${2:float y})
    "--function-arg-placeholders",

    -- (8) --fallback-style=llvm — 未找到 .clang-format 文件时的回退格式化风格
    --     可选：llvm, google, chromium, mozilla, webkit, microsoft
    "--fallback-style=llvm",

    -- (9) --query-driver — ⚠️ 仅在使用 GCC 作为编译器时需要
    --     让 clangd 调用 g++ 获取 GCC 特有的系统 include 路径
    --     例如：--query-driver=/usr/bin/g++*
    --     如果项目使用 clang/clang++ 编译，不需要此参数
  },
})
vim.lsp.enable("clangd")
```

> 来源：[nvim-lspconfig clangd 预设](https://github.com/neovim/nvim-lspconfig/blob/master/lsp/clangd.lua) | [LazyVim clangd extra](https://github.com/LazyVim/LazyVim/blob/main/lua/lazyvim/plugins/extras/lang/clangd.lua)

### 5.3 验证 LSP 是否正常

```bash
# 打开一个 cpp 文件后执行
:LspInfo                # 应显示 "clangd" 已附加
:checkhealth lsp        # 诊断 LSP 状态
```

如果显示 "clangd: no compile_commands.json found"——回到步骤 2，确认符号链接已创建。

---

## 6. 代码格式化（clang-format）

### 6.1 配置 conform.nvim

```lua
-- lua/plugins/lsp.lua 中 conform 的 formatters_by_ft 追加
-- ╔══════════════════════════════════════════════╗
-- ║ conform.nvim 基础配置见：                     ║
-- ║ → 03-lsp/formatting-linting.md §2           ║
-- ╚══════════════════════════════════════════════╝

formatters_by_ft = {
  c = { "clang-format" },       -- (10) C 语言（conform 推荐用 clang-format，旧名 clang_format 已弃用）
  cpp = { "clang-format" },     -- (11) C++ 语言
  -- 其他语言保持不变...
},
```

### 6.2 创建 .clang-format 配置文件

在项目根目录放置 `.clang-format`：

```yaml
# .clang-format — clang-format 风格配置
BasedOnStyle: LLVM           # (12) 基于 LLVM 风格
IndentWidth: 4               # (13) 缩进 4 个空格
UseTab: Never                # (14) 永不使用 Tab
ColumnLimit: 100             # (15) 每行最多 100 字符
AllowShortFunctionsOnASingleLine: Empty  # (16) 只有空函数可以放一行
AccessModifierOffset: -4     # (17) public/private 缩进偏移
```

> 详细选项：[clang-format 官方文档](https://clang.llvm.org/docs/ClangFormatStyleOptions.html)

---

## 7. 静态分析（clang-tidy）

### 7.1 配置 nvim-lint

```lua
-- lua/plugins/lsp.lua 中 nvim-lint 的 linters_by_ft 追加
-- ╔══════════════════════════════════════════════╗
-- ║ nvim-lint 基础配置见：                        ║
-- ║ → 03-lsp/formatting-linting.md §4           ║
-- ╚══════════════════════════════════════════════╝

lint.linters_by_ft = {
  c = { "clangtidy" },         -- (18) C 语言
  cpp = { "clangtidy" },       -- (19) C++ 语言
}
```

### 7.2 创建 .clang-tidy 配置文件

```yaml
# .clang-tidy — clang-tidy 静态分析规则
Checks: >
  -*,
  bugprone-*,
  modernize-*,
  performance-*,
  readability-*,
  -modernize-use-trailing-return-type
# (20) Checks — 启用的规则组
#      -* = 禁用所有默认规则
#      bugprone-* = 所有 bug 预防规则（空指针、除零等）
#      modernize-* = 现代化建议（nullptr 代替 NULL、auto 等）
#      performance-* = 性能优化建议
#      readability-* = 可读性改进
#      -modernize-use-trailing-return-type = 排除尾置返回类型建议
```

---

## 8. 从 Neovim 中构建项目

### 8.1 使用 Toggleterm（最简单）

```lua
-- lua/plugins/editor.lua
{
  "akinsho/toggleterm.nvim",
  keys = {
    { "<leader>tt", "<cmd>ToggleTerm<CR>", desc = "打开终端" },
  },
  opts = {
    direction = "float",     -- 浮动窗口
    size = function(term)
      if term.direction == "horizontal" then return 15
      elseif term.direction == "vertical" then return vim.o.columns * 0.4
      end
    end,
  },
}

-- 然后在终端中手动执行：
-- cmake --build build -j$(nproc) && ./build/myapp
```

### 8.2 使用 cmake-tools.nvim（更集成）

```lua
-- lua/plugins/lang/cpp.lua — C++ 专用插件
{
  "Civitasv/cmake-tools.nvim",
  keys = {
    { "<leader>bc", "<cmd>CMakeGenerate<CR>", desc = "CMake Configure" },
    { "<leader>bb", "<cmd>CMakeBuild<CR>", desc = "CMake Build" },
    { "<leader>br", "<cmd>CMakeRun<CR>", desc = "Run Target" },
    { "<leader>bd", "<cmd>CMakeDebug<CR>", desc = "Debug Target" },
    { "<leader>bt", "<cmd>CMakeTest<CR>", desc = "Run Tests" },
  },
  opts = {
    cmake_build_directory = "build",   -- 构建目录
    cmake_compile_commands_options = {
      action = "soft_link",            -- 自动创建 compile_commands.json 符号链接
    },
  },
}
```

| 快捷键 | 命令 | 作用 |
|--------|------|------|
| `<leader>bc` | `:CMakeGenerate` | 运行 `cmake -B build` 配置项目 |
| `<leader>bb` | `:CMakeBuild` | 运行 `cmake --build build` 编译 |
| `<leader>br` | `:CMakeRun` | 运行编译产物 |
| `<leader>bd` | `:CMakeDebug` | 以调试模式启动 |
| `<leader>bt` | `:CMakeTest` | 运行 ctest |

> 来源：[cmake-tools.nvim 文档](https://github.com/Civitasv/cmake-tools.nvim)

### 8.3 备选方案：overseer.nvim（任务运行器）

`overseer.nvim` 是一个通用任务运行器。cmake-tools 可以配置为使用 overseer 作为 executor/runner：

```lua
-- cmake-tools 使用 overseer 输出构建日志
opts = {
  cmake_executor = { name = "overseer" },
  cmake_runner = { name = "overseer" },
}
```

overseer 的优势：可以定义任意构建任务模板（不限于 CMake），支持 `.vscode/tasks.json`、`preLaunchTask`（调试前自动构建）。

### 8.4 CMake LSP（编辑 CMakeLists.txt 时的智能补全）

**neocmakelsp** 是 2025+ 年推荐的 CMake 专用 LSP（Rust + Tree-sitter），替代已废弃的 `cmake-language-server`：

```lua
-- lua/config/lsp.lua 中追加
-- (21) neocmakelsp — 为 CMakeLists.txt 提供补全、格式化、lint
--      需要系统安装：cargo install neocmakelsp（或通过 Mason）
vim.lsp.config("neocmake", {
  cmd = { "neocmakelsp", "stdio" },      -- stdio 通信模式
  filetypes = { "cmake" },              -- 仅对 cmake 文件类型生效
  root_markers = { "CMakeLists.txt", ".git" },
  init_options = {
    format = { enable = true },          -- 启用自动格式化
    lint = { enable = true },            -- 启用实时错误检测
    scan_cmake_in_package = true,        -- 扫描子目录的 CMakeLists.txt
  },
})
vim.lsp.enable("neocmake")
```

> ⚠️ 注意：旧的 `cmake-language-server`（regen100/cmake-language-server）已被 nvim-lspconfig 标记为 **DEPRECATED**，请使用 neocmakelsp。  
> 来源：[neocmakelsp README](https://github.com/neocmakelsp/neocmakelsp)

---

## 9. 调试 C++ 程序（Codelldb）

### 9.1 配置 nvim-dap

```lua
-- lua/config/dap/cpp.lua — C++ 调试器配置
local dap = require("dap")

-- (22) 适配器：CodeLLDB — 基于 LLDB 的 C/C++/Rust 调试器
dap.adapters.codelldb = {
  type = "server",                -- 以 TCP 服务器模式运行
  port = "${port}",               -- nvim-dap 自动分配端口
  executable = {
    -- (23) Mason 安装的 codelldb 适配器入口
    --      如果用系统包管理器安装，路径可能为 /usr/bin/codelldb
    command = vim.fn.stdpath("data")
      .. "/mason/packages/codelldb/extension/adapter/codelldb",
    args = { "--port", "${port}" },
  },
}

-- (24) 为 C 和 C++ 文件类型配置相同的调试行为
for _, lang in ipairs({ "c", "cpp" }) do
  dap.configurations[lang] = {
    {
      type = "codelldb",
      request = "launch",
      name = "Launch file",

      -- (25) program — 被调试的可执行文件
      --      ${workspaceFolder} = 项目根目录
      --      假设编译产物在 build/ 下，文件名与项目名一致
      --      也可以用 vim.fn.input() 手动指定
      program = function()
        return vim.fn.input(
          "Path to executable: ",
          vim.fn.getcwd() .. "/build/",
          "file"
        )
      end,
      cwd = "${workspaceFolder}",
      stopOnEntry = false,       -- (26) 不在 main() 第一条停下

      -- (27) args — 传给被调试程序的命令行参数（可选）
      -- args = { "--verbose", "--config=dev.json" },
    },
    {
      -- (28) attach 模式：连接到已运行的进程
      type = "codelldb",
      request = "attach",
      name = "Attach to process",
      pid = require("dap.utils").pick_process,  -- 弹出进程选择列表
      cwd = "${workspaceFolder}",
    },
  }
end
```

### 9.2 Debug 模式编译

```bash
# 必须使用 Debug 或 RelWithDebInfo 构建类型，否则无调试符号
cmake -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build -j$(nproc)
```

**构建类型对调试的影响**：

| Build Type | 优化级别 | 调试符号 | 适用场景 |
|------------|---------|---------|----------|
| `Debug` | `-O0 -g` | ✅ 完整 | 日常调试 |
| `RelWithDebInfo` | `-O2 -g` | ✅ 完整 | 性能敏感场景的调试 |
| `Release` | `-O3` | ❌ 无 | 生产发布，不可调试 |

### 9.3 调试操作速查

| 按键 | 操作 | 说明 |
|------|------|------|
| `<F5>` | 启动/继续 | `dap.continue()` |
| `<F9>` | 切换断点 | `dap.toggle_breakpoint()` |
| `<F10>` | 逐过程 | `dap.step_over()` |
| `<F11>` | 逐语句 | `dap.step_into()` |
| `<F12>` | 跳出 | `dap.step_out()` |
| `<leader>dr` | 打开 REPL | 交互式表达式求值 |
| `<leader>du` | 切换调试 UI | nvim-dap-ui 侧边栏 |

> ╔══════════════════════════════════════════════╗
> ║ DAP 核心概念和 UI 配置详见：                     ║
> ║ → 08-dap/dap.md                              ║
> ╚══════════════════════════════════════════════╝

---

## 10. 日常开发闭环

配置完成后，你的日常 C++ 开发循环：

```
┌──────────────────────────────────────────────────┐
│                                                    │
│  1. 写代码                                         │
│     ├── 编辑 CMakeLists.txt 时 neocmakelsp 提供补全  │
│     ├── 输入时 clangd 提供补全 (blink.cmp)           │
│     ├── 实时显示编译错误 (clangd diagnostics)        │
│     └── 保存时自动格式化 (conform + clang-format)    │
│                                                    │
│  2. 构建                           <leader>bb      │
│     ├── cmake --build build                        │
│     └── 查看编译输出（终端 / quickfix）              │
│                                                    │
│  3. 运行                           <leader>br      │
│     └── ./build/myapp [args...]                    │
│                                                    │
│  4. 调试                           <F5>            │
│     ├── 设置断点 → 启动调试                          │
│     ├── 单步执行 + 查看变量 (dap-ui)                 │
│     └── REPL 中计算表达式                           │
│                                                    │
│  5. Git 版本控制                    <leader>gs      │
│     ├── gitsigns 行内 diff 标记                     │
│     ├── neogit 暂存/提交/推送                       │
│     └── diffview 查看变更/历史                      │
│                                                    │
│  → 回到步骤 1，继续迭代                              │
│                                                    │
└──────────────────────────────────────────────────┘
```

---

## 11. 常见问题排查

### clangd 找不到头文件

```bash
# 1. 确认 compile_commands.json 存在
ls -la compile_commands.json

# 2. 检查符号链接
readlink compile_commands.json

# 3. 查看 clangd 日志
:LspLog

# 4. 如果使用 GCC 编译器，添加 --query-driver
vim.lsp.config("clangd", {
  cmd = { "clangd", "--query-driver=/usr/bin/g++" }
})
```

### 补全/跳转定义不工作

```vim
:LspInfo          " 检查 clangd 是否已附加
:LspRestart       " 重启 LSP 服务器
```

### build/compile_commands.json 未生成

```cmake
# 确认 CMakeLists.txt 中有这一行
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
```

### codelldb 启动失败

```bash
# 确认 Mason 已安装 codelldb
:Mason             # 搜索 codelldb，确认已安装

# 或通过系统包管理器安装
sudo apt install codelldb
```

---

## 12. 关键参考

| 资源 | URL |
|------|-----|
| clangd 官方文档 | https://clangd.llvm.org/ |
| clangd compile_commands | https://clangd.llvm.org/design/compile-commands |
| CMake 官方文档 | https://cmake.org/documentation/ |
| nvim-lspconfig clangd | https://github.com/neovim/nvim-lspconfig/blob/master/lsp/clangd.lua |
| LazyVim clangd extra | https://github.com/LazyVim/LazyVim/blob/main/lua/lazyvim/plugins/extras/lang/clangd.lua |
| cmake-tools.nvim | https://github.com/Civitasv/cmake-tools.nvim |
| clang-format 选项 | https://clang.llvm.org/docs/ClangFormatStyleOptions.html |
| clang-tidy 规则列表 | https://clang.llvm.org/extra/clang-tidy/checks/list.html |
