# 12-cpp-workflow — C++ 开发全流程实战

## OVERVIEW
C++/CMake/Git 全流程实战教程（1 文件，571 行，全项目最大单文件）。以 step-by-step 方式串联本篇所有章节：LSP（clangd）→ 构建（CMake/cmake-tools）→ 格式化（clang-format）→ 静态分析（clang-tidy）→ 调试（codelldb）→ Git。

## STRUCTURE
```
12-cpp-workflow/
└── README.md    # 9 个步骤的完整流程：从系统依赖到日常开发闭环
```

## WHERE TO LOOK
| Task | File | Section |
|------|------|---------|
| compile_commands.json 是什么/为什么 | `README.md` | 步骤 1 |
| 最小 CMakeLists.txt 逐行解释 | `README.md` | 步骤 2 |
| clangd 的 8 个启动参数详解 | `README.md` | 步骤 3 |
| .clang-format 文件编写 | `README.md` | 步骤 4 |
| .clang-tidy 规则组选择 | `README.md` | 步骤 5 |
| cmake-tools.nvim 配置 | `README.md` | 步骤 6 |
| neocmakelsp（CMake 专用 LSP）配置 | `README.md` | 步骤 6.4 |
| codelldb 调试器配置 | `README.md` | 步骤 7 |
| Debug/Release 构建类型对调试的影响 | `README.md` | 步骤 7.2 |
| 日常开发闭环流程图 | `README.md` | 步骤 8 |
| clangd 找不到头文件的排查 | `README.md` | 步骤 9 |

## CONVENTIONS (THIS DIR)
- 每个步骤都链接回对应教程章节（如「╔══ DAP 核心概念详见 08-dap/dap.md ══╗」）
- 新概念（compile_commands.json、CMakePresets、neocmakelsp）在文中首次出现时给出完整解释
- 代码块使用逐句编号注解（与 03-lsp/、04-completion/ 风格一致）
- 前置阅读表明确列出了读者必须先完成的其他章节

## ANTI-PATTERNS
- 不要混用 `cmake-language-server`（已废弃）—— 使用 `neocmakelsp`
- 不要用 Release 模式编译后尝试调试——必须用 Debug 或 RelWithDebInfo
- 不要忘记 `ln -sf build/compile_commands.json .`——clangd 大多数情况下无法自动找到 build 子目录
