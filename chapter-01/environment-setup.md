# 1.2 开发环境搭建

编写 C++ 程序需要两样基本工具：一个**编译器**（Compiler）用于将源代码转换为可执行程序，一个**编辑器**或**集成开发环境**（IDE, Integrated Development Environment）用于编写代码。本节介绍主流平台下的环境搭建方式。

## 编译器的选择

目前主流的 C++ 编译器有三个：

| 编译器 | 平台 | 说明 |
|--------|------|------|
| GCC (g++) | Linux / macOS / Windows | GNU 编译器集合，开源，标准支持全面 |
| Clang (clang++) | Linux / macOS / Windows | LLVM 项目的编译器，错误提示友好，编译速度快 |
| MSVC (cl.exe) | Windows | 微软 Visual C++ 编译器，与 Visual Studio 深度集成 |

三者都能良好支持 C++17 标准。初学阶段选择任意一个即可。

## Windows 环境

### 方案一：Visual Studio（推荐）

Visual Studio 是 Windows 上最成熟的 C++ 开发环境，自带 MSVC 编译器、调试器和项目管理工具。

1. 访问 Visual Studio 官网，下载 **Community** 版本（免费）。
2. 运行安装程序，在工作负载（Workloads）页面勾选「使用 C++ 的桌面开发」。
3. 等待安装完成。

安装后，打开 Visual Studio，选择「创建新项目」→「控制台应用」即可开始编写代码。

### 方案二：MinGW-w64 + 编辑器

如果你更喜欢轻量级的开发方式，可以单独安装 GCC 编译器：

1. 下载 MinGW-w64（推荐通过 MSYS2 安装）。
2. 安装 MSYS2 后，在其终端中执行以下命令安装 GCC：

```bash
pacman -S mingw-w64-ucrt-x86_64-gcc
```

3. 将编译器路径添加到系统环境变量 `PATH` 中。
4. 打开命令提示符，验证安装：

```bash
g++ --version
```

如果输出了版本号信息，说明安装成功。

## Linux 环境

大多数 Linux 发行版的软件仓库中已包含 GCC。以 Ubuntu / Debian 为例：

```bash
sudo apt update
sudo apt install g++ build-essential
```

验证安装：

```bash
g++ --version
```

如果需要 Clang，可以额外安装：

```bash
sudo apt install clang
```

## macOS 环境

macOS 可以通过 Xcode 命令行工具获取 Clang 编译器：

```bash
xcode-select --install
```

安装完成后验证：

```bash
clang++ --version
```

如果需要 GCC，可以通过 Homebrew 安装：

```bash
brew install gcc
```

## 编辑器与 IDE

除 Visual Studio 外，以下编辑器也适合 C++ 开发：

- **Visual Studio Code** — 微软出品的轻量级编辑器，安装 C/C++ 扩展后可获得代码补全、调试等功能。跨平台，免费。
- **CLion** — JetBrains 出品的专业 C++ IDE，功能强大，适合中大型项目。付费软件，学生可申请免费许可。
- **Vim / Neovim** — 终端编辑器，配合 LSP 插件可实现高效的 C++ 开发，学习曲线较陡。

初学阶段建议使用 **Visual Studio**（Windows）或 **Visual Studio Code**（跨平台），上手简单，遇到问题也容易查找资料。
