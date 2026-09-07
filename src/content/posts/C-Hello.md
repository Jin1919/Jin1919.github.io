---
title: 如何在vscode中写一个C++程序？
date: 2026-09-07T12:45:27.000Z
tags: [C++]
category: 教程
comments: true
draft: false
---

> 本文以简单的 `Hello World` 程序为例，讲解 VS Code 下 C++ 的安装、配置、编译、运行与调试完整流程。
> VS Code 只是代码编辑器，**本身不自带 C++ 编译器**，需要额外安装编译器 + 配套扩展才可以正常编写运行 C++ 代码。

## 一、安装 VS Code

前往 [VS Code 官网](https://code.visualstudio.com/) 下载并安装 Visual Studio Code。

> 安装时建议勾选以下选项：

- 将 “通过 Code 打开” 添加到文件右键菜单
- 将 VS Code 添加到 PATH 环境变量
- 将 VS Code 注册为受支持的文件类型编辑器

## 二、安装 C++ 编译器

### Windows（MSYS2 环境）

本文使用 MSYS2 作为编译环境。MSYS2 提供完整的 GCC 工具链，相比 MinGW‑w64 更新、包管理更方便。

1. 前往 [MSYS2官网](https://www.msys2.org/) 下载安装 MSYS2。
2. 打开 **MSYS2 UCRT64** 终端（必须是 UCRT64，不要用 MSYS、MINGW64），执行更新系统并安装编译工具链：

```bash
pacman -Syu
pacman -S mingw-w64-ucrt-x86_64-gcc mingw-w64-ucrt-x86_64-gdb
```

3. 将 MSYS2 的 UCRT64 的 `bin` 目录添加到系统环境变量 `Path`
   示例路径（根据你的安装目录修改）：

```
C:\msys64\ucrt64\bin
```

> ⚠️ 注意：不要添加 msys2/usr/bin，只添加 `ucrt64/bin`。

4. **重启 PowerShell / Windows终端**，验证编译器是否生效

```
g++ --version
gdb --version
```

输出版本信息即配置成功。

### Linux（Ubuntu）

```
sudo apt update
sudo apt install build-essential gdb
# 验证
g++ --version
```

### macOS

安装 Xcode 命令行工具

```
xcode-select --install
# 验证
clang++ --version
```

## 三、安装 VS Code 扩展

打开左侧扩展面板（快捷键 `Ctrl+Shift+X`），搜索安装：

1. **C/C++（Microsoft）**：语法高亮、智能补全、调试核心扩展【必装】
2. **Code Runner**：可选，快速一键运行小代码片段
   安装完成后重启 VS Code。

## 四、创建 C++ 项目

1. 新建文件夹 `cpp‑demo`，用 VS Code 打开该文件夹
2. 在项目内新建文件 `main.cpp`

```
#include <iostream>
int main() {
    std::cout << "Hello, C++!" << std::endl;
    return 0;
}
```

**代码说明**

- `#include <iostream>`：引入输入输出库
- `int main()`：程序入口函数
- `std::cout`：控制台输出
- `return 0`：代表程序正常退出

## 五、编译并运行程序

### VS Code 内置终端运行

调出终端快捷键：`Ctrl + ``

> Windows终端建议使用 PowerShell，环境变量配置完成后可以直接调用 `g++`，不需要打开 MSYS2 窗口。

进入源码所在目录，执行编译：

```
g++ main.cpp -o main
```

运行程序

```
# Windows
.\main.exe
# Linux / macOS
./main
```

输出结果：

```
Hello, C++!
```

**编译运行合并一条命令**

```
# Linux/macOS
g++ main.cpp -o main && ./main
```

```
# Windows PowerShell
g++ main.cpp -o main.exe
if ($LASTEXITCODE -eq 0) { .\main.exe }
```

### 使用 Code Runner 运行（适合简单小代码）

1. 打开 `main.cpp`
2. 点击右上角运行按钮，或者右键代码区选择 `Run Code`
3. 输出结果展示在输出面板>

> ⚠️ 注意：Code Runner 适合简单 Demo；做调试、输入交互、多文件项目，优先使用内置终端。

## 六、调试功能配置

1. 在代码行号左侧点击添加**断点**
2. 左侧切换到「运行和调试」面板，选择 `C++ (GDB/LLDB)`
3. 首次调试会自动生成 `.vscode` 配置文件夹

> 编译时带上 `-g` 参数生成调试符号信息：

```
g++ -g main.cpp -o main
```

可用调试能力：

- 单步执行
- 变量监视
- 查看调用栈
- 继续运行 / 终止调试

> MSYS2环境注意：VS Code调试会自动调用系统PATH里的gdb，确认 `ucrt64/bin` 在系统Path中，否则调试器无法启动。

## 七、配置自动编译任务 tasks.json

新建文件 `.vscode/tasks.json`，复制下面全部内容：

```
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "build cpp",
      "type": "shell",
      "command": "g++",
      "args": [
        "-g",
        "${file}",
        "-o",
        "${fileDirname}/${fileBasenameNoExtension}"
      ],
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "problemMatcher": [
        "$gcc"
      ]
    }
  ]
}
```

保存后快捷键 `Ctrl + Shift + B`，一键编译当前打开的 C++ 文件。

## 八、常见问题排查

### 1. 提示找不到 g++

- Windows：检查 MSYS2 的 `ucrt64/bin` 是否加入系统 `Path`；修改环境变量后**必须重启终端/VS Code**。

```
where g++
```

> 如果输出路径不是 `C:\msys64\ucrt64\bin\g++.exe`，代表环境变量配置错误。

- Linux / macOS

```
which g++
```

### 2. 控制台中文乱码

源文件保存为 UTF‑8 编码，终端设置使用 UTF‑8 字符集。

### 3. 运行窗口一闪而过

不要双击 exe，直接在 VS Code 终端运行程序，保留输出窗口。

### 4. 使用 C++11 / C++17 / C++20 新语法

编译时手动指定标准版本：

```
g++ main.cpp -std=c++17 -o main
g++ main.cpp -std=c++20 -o main
```

### 5. MSYS2 常见坑

1. 不要使用 MSYS2 的 MSYS 子系统，务必使用 **UCRT64**；
2. 环境变量只添加 `ucrt64/bin`，不要添加 `usr/bin`；
3. 修改系统环境变量后，所有终端、VS Code 需要完全重启才会生效。

## 总结

VS Code 编写 C++ 的完整流程：

1. 安装 VS Code
2. Windows安装MSYS2，安装ucrt64 gcc/gdb工具链并配置系统环境变量；Linux/macOS安装对应编译器
3. 安装 Microsoft C/C++ 扩展
4. 创建 `.cpp` 源码文件
5. 调用 g++ 编译源代码
6. 运行可执行程序，或开启调试

> 核心区分：VS Code = 编辑器；g++/clang = 编译器；编辑器负责写代码，编译器负责把源码转为可执行程序，分清两者可以解决绝大多数配置问题。
