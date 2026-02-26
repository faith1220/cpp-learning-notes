# 1.4 编译与运行

在上一节中，我们编写了 `hello.cpp` 源文件。但计算机无法直接执行源代码文本，需要通过**编译**将其转换为机器可执行的二进制程序。本节介绍编译与运行的基本操作，并简要说明编译过程的各个阶段。

## 命令行编译与运行

### 使用 g++（GCC）

打开终端（Windows 下为命令提示符或 PowerShell），进入 `hello.cpp` 所在目录，执行：

```bash
g++ -o hello hello.cpp
```

- `g++` 是编译器命令
- `-o hello` 指定输出文件名为 `hello`（Windows 下会生成 `hello.exe`）
- `hello.cpp` 是源文件

编译成功后不会有任何输出。然后运行程序：

```bash
# Linux / macOS
./hello

# Windows
hello.exe
```

终端将输出：

```
Hello, World!
```

### 使用 clang++（Clang）

命令格式与 g++ 几乎完全一致：

```bash
clang++ -o hello hello.cpp
```

### 指定 C++ 标准版本

默认情况下，编译器可能不会使用最新的 C++ 标准。要明确指定使用 C++17，添加 `-std=c++17` 参数：

```bash
g++ -std=c++17 -o hello hello.cpp
```

本书的示例代码基于 C++17，建议始终加上这个参数。

## 在 Visual Studio 中编译运行

如果你使用 Visual Studio：

1. 创建新项目，选择「控制台应用」模板。
2. 在自动生成的源文件中编写代码（或替换为上一节的代码）。
3. 按 `Ctrl + F5`（开始执行但不调试）或 `F5`（开始调试）运行程序。
4. 输出结果会显示在弹出的控制台窗口中。

Visual Studio 默认使用 MSVC 编译器，C++17 支持通常已在项目设置中启用。如需确认，可在项目属性 → C/C++ → 语言 → C++ 语言标准中检查。

## 编译过程的四个阶段

从源代码到可执行文件，编译器实际上经历了四个阶段。此处先做概览，第十八章将深入讨论。

### 1. 预处理（Preprocessing）

处理所有以 `#` 开头的指令。`#include` 会将头文件内容插入到源文件中，`#define` 会执行宏替换。预处理后的结果仍然是文本形式的 C++ 代码，但体积通常会大幅增加。

```bash
# 查看预处理结果（仅用于了解，日常开发不需要）
g++ -E hello.cpp -o hello.i
```

### 2. 编译（Compilation）

将预处理后的 C++ 代码翻译为**汇编代码**（Assembly Code）。这一步会进行语法检查、类型检查以及各种优化。

```bash
g++ -S hello.cpp -o hello.s
```

### 3. 汇编（Assembly）

将汇编代码转换为**目标文件**（Object File），即二进制格式的机器指令。目标文件尚不能直接运行，因为其中可能引用了外部函数（如标准库中的函数），这些引用还没有被解析。

```bash
g++ -c hello.cpp -o hello.o
```

### 4. 链接（Linking）

将一个或多个目标文件与所需的库文件合并，解析所有外部引用，生成最终的**可执行文件**（Executable）。

```bash
g++ hello.o -o hello
```

完整的编译流程可以概括为：

```
源代码 (.cpp)
  → 预处理 → 展开后的源代码 (.i)
  → 编译   → 汇编代码 (.s)
  → 汇编   → 目标文件 (.o / .obj)
  → 链接   → 可执行文件 (hello / hello.exe)
```

日常开发中，我们通常直接使用 `g++ -o hello hello.cpp` 一步完成所有阶段，无需手动拆分。

## 编译错误与警告

编译时如果代码存在语法错误，编译器会输出错误信息并终止编译。例如，故意去掉分号：

```cpp
std::cout << "Hello, World!" << std::endl  // 缺少分号
```

编译器会报告类似以下内容：

```
hello.cpp:4:49: error: expected ';' after expression
```

这条信息告诉你：`hello.cpp` 文件第 4 行第 49 列处，需要一个分号。学会阅读编译器的错误信息是 C++ 学习中非常重要的技能。

建议在编译时开启更多警告，帮助发现潜在问题：

```bash
g++ -std=c++17 -Wall -Wextra -o hello hello.cpp
```

- `-Wall` 开启大部分常见警告
- `-Wextra` 开启额外的警告检查

养成开启警告编译的习惯，可以在早期发现许多隐藏的错误。
