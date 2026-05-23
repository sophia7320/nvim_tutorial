# nvim-dap 各语言调试适配器 — 逐行深度配置

> **TL;DR**: 不同语言的 DAP 适配器配置完全不同。逐句提供 Python（debugpy）、JS/TS（vscode-js-debug）、Rust（CodeLLDB）、C++（cppdbg / CodeLLDB）、Go（Delve）、Java（nvim-jdtls + java-debug）的完整配置。

---

## 1. Python — debugpy

```lua
-- lua/config/dap/python.lua

local dap = require("dap")  -- (1) 加载 nvim-dap

-- ─── 适配器：告诉 nvim-dap 如何启动 debugpy ──────────
dap.adapters.python = {
  type = "executable",              -- (2) 启动子进程，通过 stdio 通信
  command = "python3",              -- (3) Python 解释器路径
  args = { "-m", "debugpy.adapter" }, -- (4) 以模块方式运行 debugpy 适配器
  -- 等价于 shell: python3 -m debugpy.adapter
}

-- ─── 配置：告诉 debugpy 如何运行你的代码 ──────────────
dap.configurations.python = {
  {
    type = "python",                -- (5) 必须匹配适配器键名
    request = "launch",             -- (6) 启动新进程
    name = "Launch file",           -- (7) 选择菜单显示名
    program = "${file}",            -- (8) ${file} = 当前文件路径
    console = "integratedTerminal", -- (9) 输出到 Neovim 终端 [默认值，可选]
    justMyCode = false,             -- (10) false: 可进入第三方库代码

    -- (11) pythonPath — 动态选择 Python 解释器
    pythonPath = function()
      local cwd = vim.fn.getcwd()
      if vim.fn.executable(cwd .. "/venv/bin/python") == 1 then
        return cwd .. "/venv/bin/python"    -- .venv/
      elseif vim.fn.executable(cwd .. "/.venv/bin/python") == 1 then
        return cwd .. "/.venv/bin/python"   -- ./venv/
      end
      return "python3"                      -- 系统默认
    end,
  },
}
```

---

## 2. JavaScript / TypeScript — vscode-js-debug

```lua
-- lua/config/dap/javascript.lua

local dap = require("dap")

-- (12) 适配器：server 模式 — nvim-dap 先启动 node 服务器再 TCP 连接
dap.adapters["pwa-node"] = {
  type = "server",
  host = "localhost",
  port = "${port}",            -- (13) nvim-dap 自动分配端口

  executable = {
    command = "node",
    args = {
      -- (14) Mason 安装的 js-debug DAP 服务器入口
      vim.fn.stdpath("data")
        .. "/mason/packages/js-debug-adapter/js-debug/src/dapDebugServer.js",
      "${port}",
    },
  },
}

dap.configurations.javascript = {
  {
    type = "pwa-node",
    request = "launch",
    name = "Launch file",
    program = "${file}",
    cwd = "${workspaceFolder}",
  },
  -- (15) attach 模式：连接已运行进程
  {
    type = "pwa-node",
    request = "attach",
    name = "Attach to process",
    port = 9229,               -- (16) Node.js 默认调试端口
    restart = true,            -- (17) 断开后自动重连
    sourceMaps = true,         -- (18) TS → JS source map [默认值，可选]
    protocol = "inspector",    -- (19) V8 Inspector 协议 [默认值，可选]
  },
}
-- (20) TS/TSX 复用 JS 配置
dap.configurations.typescript = dap.configurations.javascript
```

---

## 3. Rust — CodeLLDB

```lua
-- lua/config/dap/rust.lua

local dap = require("dap")

dap.adapters.codelldb = {
  type = "server",
  port = "${port}",
  executable = {
    -- (21) Mason 安装的 codelldb 适配器
    command = vim.fn.stdpath("data")
      .. "/mason/packages/codelldb/extension/adapter/codelldb",
    args = { "--port", "${port}" },
  },
}

dap.configurations.rust = {
  {
    type = "codelldb",
    request = "launch",
    name = "Launch",
    -- (22) 手动指定编译产物路径 (target/debug/<name>)
    program = function()
      return vim.fn.input(
        "Path to executable: ",
        vim.fn.getcwd() .. "/target/debug/",
        "file"
      )
    end,
    cwd = "${workspaceFolder}",
    stopOnEntry = false,       -- (23) 不在入口停下
  },
}
```

---

## 4. Go — Delve

```lua
-- lua/config/dap/go.lua

local dap = require("dap")

dap.adapters.delve = {
  type = "server",
  port = "${port}",
  executable = {
    command = "dlv",
    -- (24) dlv dap 启动 DAP 服务器
    args = { "dap", "-l", "127.0.0.1:${port}" },
  },
}

dap.configurations.go = {
  {
    type = "delve",
    request = "launch",
    name = "Debug file",
    program = "${file}",
  },
  -- (25) 调试测试
  {
    type = "delve",
    request = "launch",
    name = "Debug test",
    mode = "test",
    program = "${workspaceFolder}",
  },
}
```

---

## 5. C++ — cppdbg (GDB) / CodeLLDB

> ╔══════════════════════════════════════════════╗
> ║ C++ 项目的完整调试流程（含 CMake 构建集成）     ║
> ║ 详见 → 12-cpp-workflow/README.md §9           ║
> ╚══════════════════════════════════════════════╝

C++ 有两种主流 DAP 适配器：**CodeLLDB**（LLDB 前端，支持 C/C++/Rust/Swift）和 **cppdbg**（Microsoft 的 GDB/LLDB 调试器）。Rust 章节已配置 CodeLLDB（§3），C++ 可直接复用。此处展示 GDB 方案。

```lua
-- lua/config/dap/cpp.lua
local dap = require("dap")

-- ─── 适配器：cppdbg（Microsoft C/C++ 调试器）────────────────
-- (26) Mason 包名：cpptools → DAP 适配器名：cppdbg
--      type = "executable" 表示启动本地可执行文件作为适配器
dap.adapters.cppdbg = {
  type = "executable",
  -- (27) Mason 安装的 OpenDebugAD7 入口（GDB/LLDB 前端）
  command = vim.fn.stdpath("data")
    .. "/mason/packages/cpptools/extension/debugAdapters/bin/OpenDebugAD7",
}

-- ─── 配置：Launch 和 Attach ───────────────────────────────
dap.configurations.cpp = {
  {
    -- (28) launch — 直接启动可执行文件
    name = "Launch file (GDB)",
    type = "cppdbg",
    request = "launch",
    -- (29) program — 动态指定编译产物路径
    program = function()
      return vim.fn.input(
        "Path to executable: ",
        vim.fn.getcwd() .. "/build/",
        "file"
      )
    end,
    cwd = "${workspaceFolder}",
    stopAtEntry = true,       -- (30) 在 main() 入口暂停，方便观察初始化
  },
  {
    -- (31) 带命令行参数启动
    name = "Launch with args",
    type = "cppdbg",
    request = "launch",
    program = function()
      return vim.fn.input("Executable: ", vim.fn.getcwd() .. "/build/", "file")
    end,
    args = function()
      return vim.split(vim.fn.input("Args: "), " +", { trimempty = true })
    end,
    cwd = "${workspaceFolder}",
    stopAtEntry = false,
  },
  {
    -- (32) Attach 到远程 gdbserver（嵌入式 / 远程调试）
    name = "Attach to gdbserver",
    type = "cppdbg",
    request = "launch",
    MIMode = "gdb",
    miDebuggerServerAddress = function()
      return vim.fn.input("[host]:port : ", "localhost:7777")
    end,
    miDebuggerPath = vim.fn.exepath("gdb"),  -- (33) 系统 gdb 路径
    cwd = "${workspaceFolder}",
    program = function()
      return vim.fn.input("Symbol file: ", vim.fn.getcwd() .. "/build/", "file")
    end,
  },
}
-- (34) C 语言复用同一套配置
dap.configurations.c = dap.configurations.cpp
```

**如果使用 CodeLLDB 方案**（复用 §3 的配置）：
```lua
dap.configurations.cpp = {
  { type = "codelldb", request = "launch", name = "Launch (LLDB)",
    program = function() return vim.fn.input("Executable: ", vim.fn.getcwd() .. "/build/", "file") end,
    cwd = "${workspaceFolder}", stopOnEntry = false,
  },
}
dap.configurations.c = dap.configurations.cpp
```

> ⚠️ 无论哪种方案，编译时**必须使用 Debug 模式**：
> ```bash
> cmake -B build -DCMAKE_BUILD_TYPE=Debug && cmake --build build
> ```

---

## 6. Java — nvim-jdtls + java-debug

Java 调试适配器**不是独立的**——它是 `eclipse.jdt.ls`（jdtls）的插件扩展。因此必须通过 `nvim-jdtls` 加载并注册到 `nvim-dap`。

**Mason 需要安装**：`java-debug-adapter` + `java-test`

```lua
-- lua/config/dap/java.lua
-- 注意：此文件可放在 ftplugin/java.lua 中，仅在打开 Java 文件时执行

-- (35) 获取 Mason 安装的调试扩展路径
local mason_registry = require("mason-registry")
local java_dbg_path = mason_registry.get_package("java-debug-adapter"):get_install_path()
local java_test_path = mason_registry.get_package("java-test"):get_install_path()

-- (36) 收集需要注入 jdtls 的 bundle jar 包
local bundles = {
  -- java-debug 核心 — com.microsoft.java.debug.plugin-*.jar
  vim.fn.glob(java_dbg_path .. "/extension/server/com.microsoft.java.debug.plugin-*.jar", 1),
}

-- (37) 注入 java-test 相关 jar（排除 runner 和 jacoco）
local test_jars = vim.split(
  vim.fn.glob(java_test_path .. "/extension/server/*.jar", 1), "\n"
)
for _, jar in ipairs(test_jars) do
  local name = vim.fn.fnamemodify(jar, ":t")
  if name ~= "com.microsoft.java.test.runner-jar-with-dependencies.jar"
    and name ~= "jacocoagent.jar" then
    table.insert(bundles, jar)
  end
end

-- (38) jdtls 启动配置 — 将 bundles 注入到 init_options 中
local jdtls = require("jdtls")
local config = {
  cmd = { "jdtls" },
  root_dir = vim.fs.root(0, { "gradlew", ".git", "mvnw", "pom.xml" }),
  init_options = {
    bundles = bundles,  -- (39) 🔑 关键：通过此字段加载调试扩展
  },
  settings = { java = {} },
}
jdtls.start_or_attach(config)

-- (40) 关键：启用 DAP 集成（自动注册 dap.adapters.java）
--      必须等 jdtls 启动完成后调用
jdtls.setup_dap({ hotcodereplace = "auto" })

-- (41) 自动发现项目中的 Main Class，生成 launch 配置
require("jdtls.dap").setup_dap_main_class_configs()

-- ─── 手动 Attach 配置（远程调试）───────────────────────────
-- (42) 用于连接已运行的 JVM（JVM 启动参数：-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005）
local dap = require("dap")
dap.configurations.java = dap.configurations.java or {}
table.insert(dap.configurations.java, {
  type = "java",
  request = "attach",
  name = "Attach Remote JVM",
  hostName = "127.0.0.1",
  port = 5005,
})

-- ─── 测试调试快捷键 ──────────────────────────────────────
-- (43) 调试当前测试类 / 最近的测试方法
vim.keymap.set("n", "<leader>dt", function()
  require("jdtls").test_class()
end, { desc = "DAP: Debug Test Class" })
vim.keymap.set("n", "<leader>dn", function()
  require("jdtls").test_nearest_method()
end, { desc = "DAP: Debug Nearest Test" })
```

**与 jdtls LSP 的集成结构**：

```text
┌────────────────────────────────────┐
│           nvim-jdtls               │
│  ┌──────────────────────────────┐  │
│  │  jdtls.start_or_attach()     │  │  ← 启动 LSP
│  │    init_options.bundles = {  │  │
│  │      java-debug-adapter,     │  │  ← 注入调试器
│  │      java-test,              │  │  ← 注入测试运行器
│  │    }                         │  │
│  └──────────────────────────────┘  │
│  ┌──────────────────────────────┐  │
│  │  jdtls.setup_dap()           │  │  ← 注册到 nvim-dap
│  │    → dap.adapters.java       │  │
│  └──────────────────────────────┘  │
└────────────────────────────────────┘
```

---

## 7. CMake 项目调试

**CMake 本身没有独立的调试适配器**。CMake 项目调试 = 用 C++ 适配器（CodeLLDB 或 cppdbg）调试 CMake 构建出的可执行文件。

### 推荐方案：cmake-tools.nvim 集成

`cmake-tools.nvim` 提供 `:CMakeDebug` 命令，自动调用 `nvim-dap`：

```lua
-- lua/plugins/lang/cpp.lua 中 cmake-tools 的 DAP 配置
opts = {
  cmake_build_directory = "build",

  -- (44) 指定调试时使用的 DAP 配置
  cmake_dap_configuration = {
    name = "CMake Debug",
    type = "codelldb",          -- (45) 复用 Rust/C++ 的 CodeLLDB 适配器
    request = "launch",
    stopOnEntry = false,
    runInTerminal = true,       -- (46) 在集成终端运行，支持 stdin 交互
  },
}
```

**使用流程**：`:CMakeSelectBuildType Debug` → `:CMakeBuild` → `:CMakeDebug`

### 手动方案（不依赖 cmake-tools）

在 C++ 的 `dap.configurations.cpp` 中添加一个 CMake 专用配置（见 §5），使用 `codelldb` 或 `cppdbg` 均可。

> ⚠️ **Debug 模式是硬性前提**。Release 模式不含调试符号，无法设置断点。
> ```bash
> cmake -B build -DCMAKE_BUILD_TYPE=Debug && cmake --build build
> ```
>
> ╔══════════════════════════════════════════════╗
> ║ C++/CMake 全流程（LSP→构建→调试→Git）        ║
> ║ 详见 → 12-cpp-workflow/README.md §8-9       ║
> ╚══════════════════════════════════════════════╝

---

## 8. 关键参考

| 资源 | URL |
|------|-----|
| nvim-dap 文档 | https://github.com/mfussenegger/nvim-dap/blob/master/doc/dap.txt |
| Debug Adapter Wiki | https://codeberg.org/mfussenegger/nvim-dap/wiki/Debug-Adapter-installation |
| nvim-dap-python | https://github.com/mfussenegger/nvim-dap-python |
| mason-nvim-dap 映射表 | https://github.com/jay-babu/mason-nvim-dap.nvim |
| nvim-jdtls (Java DAP) | https://github.com/mfussenegger/nvim-jdtls#debugger-via-nvim-dap |
| cmake-tools.nvim DAP | https://github.com/Civitasv/cmake-tools.nvim |
