# 18.3 CMake 基础

在前两节我们了解到，编译链接一个 C++ 项目绝非易事。如果项目只有两三个文件，你还可以勉强在黑盒黑窗口命令行里敲出：
`g++ main.cpp math.cpp utils.cpp -o app.exe`

但假如你的项目包含了分布在七八个深层级文件夹下的 150 多个 cpp 源文件，并且有些代码还得依赖引入外部下载在某处的 OpenCV 图像库、底层还得链接操作系统专用的网络库包。你要怎么告诉编译器这繁重的一切？

这时我们需要用到**构建系统（Build System）**。由于 C++ 跨平台差异过大，历史上群雄割据混乱不堪，大家受够了这种折磨。后来一位名叫 **CMake** 的王者横空出世统一步伐。如今的 CMake 已经是事实上的 C++ 工业界绝对的霸主与标准（类似于前端的 npm，Java 的 Maven/Gradle，Rust 的 Cargo）。

绝大部分的现代 IDE（包括 CLion，甚至微软巨兽 Visual Studio），都是原生地全面围绕接纳直接使用 CMake 为组织工程命脉进行支持。

## 什么是 CMake？CMakeLists.txt

CMake 其实是一个**生成器的生成器（Meta-Build System）**。

你只需要在一个叫做 `CMakeLists.txt` 的纯文本剧本里，用极其简单的、高度高度抽象的声明向 CMake 描述：“我的项目叫啥，要用 C++ 新的第几版标准，要纳入把哪些文件捏在一起变出来一个叫什么名字的执行程序”。

只要打出 `cmake` 命令：
- 在 Windows 下，CMake 自己会把这一套看懂，生成一整个繁杂晦长恶心的 Visual Studio 项目底层 `.sln/.vcxproj` 让 MSVC 编译器干活。
- 在 Linux 下，CMake 给你看懂，替你手写出成百上千行恶心晦涩的 `Makefile` 给 Make 引擎调用 gcc 干活。

## 最简单的 CMakeLists 剧本骨架

想要管理你自己的第一个基于 CMake 运作的正规 C++ 工程，你可以在项目的最顶层根目录下创设一个叫 `CMakeLists.txt` 的文本文档并照抄如下的四大必备起手式：

```cmake
# 声明1：规定使用 CMake 工具的最低能够看懂本语法的版本限制
cmake_minimum_required(VERSION 3.10)

# 声明2：给你的整组宏大工程设立总名称
project(MySuperApp)

# 声明3：最关键！要求全项目强制采用 C++17 的语法标准规格进行预备编译
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE) # 如果系统里没 C++17 的编译工具直接掐断退出别继续往下走废功夫了

# 声明4：发号施令做苦力。向目标叫 "app_run" 的那个即将喷出的 exe 二进制里，融合塞入这些并排的所有的 cpp 源文件参与编译链接！
add_executable(app_run main.cpp src/math.cpp src/helper.cpp)
```

有了这份 `CMakeLists.txt`，你只要在含有它的这个目录打开终端：
1. `cmake -B build` （在旁边创立个全新的空抽屉抽屉包 `build` 放杂物垃圾，接着生成定制给你的全套构建环境）
2. `cmake --build build` （彻底拉动底层引擎马达开始根据图纸全线编译并融和链接）

完毕！到抽屉包里去找一个能完美双击运行的成品出产即可！

## CMake 如何管理与链接库？

这绝对是每个新人学 C++ 最后被反复挫败折磨的环节：我们想加入别人的成熟类库为我们所用（如用到一个物理加速库叫 `PhysicsDB`）。

你必须用 CMake 替你搞清楚，并指引给底层链接器告诉两件生死攸关的寻路大事令：
1. **去哪找它的头文件 `.h`（以便让当前代码的预处理能找准位置做声明）**
2. **去哪找汇编编译过的成品机器码躯壳 `.lib / .a / .so`（以便让链接器在第四站能完美结网对接上填上调用空穴）**

在非常标准的工程结构（被引入到了你机器里的系统预设安装架构的话），在 CMakeLists 中使用大杀器只需三步极其无极轻松拿捏引入库容：

```cmake
# 依旧前四步常规
cmake_minimum_required(VERSION 3.10)
project(GameEngine)
set(CMAKE_CXX_STANDARD 17)

# 把你的目标核心代码纳入编译列队
add_executable(game main.cpp hero.cpp)

# 【库引入的核心三板斧开始！】
# 第 1 步：找库。这叫 find 大法。CMake 会在全系统自动翻找名为 "OpenCV" 这大名鼎鼎的包裹以及它身上挂的所有附属预设路径数据！
find_package(OpenCV REQUIRED)

# 第 2 步：包引入。为刚刚你的那个可怜的小 "game" 躯体，指出一条明路让他能在包含如 `#include <opencv2/core.hpp>` 预处理展开时找到路径位置找对目标
target_include_directories(game PRIVATE ${OpenCV_INCLUDE_DIRS})

# 第 3 步：缝躯体。也是绝对成败的一环，强制把库底层的实现机器码 `.lib` 与 "game" 的残端缝连填埋链接坑洞结合！
target_link_libraries(game PRIVATE ${OpenCV_LIBS})
```

至此，你能完全独自一人从零用 CMake 搭建拉扯出一座带有图形界面交互或是具有强力网络请求处理等一切第三方现代巨擘支撑的现代大型化企业级别 C++ 控制台应用地基！

**恭喜！这是本套完整学习资料的最后一节！**
当你能够真正理解预处理与链接机制的区别、明确理解 `#pragma once` 和多重定义的避免机制，使用结构化清晰分离了海量的代码架构并且成功靠寥寥 10 行神妙的 `CMake` 把外部强力库融合成一个独属于你的 `.exe` 程序时——

**你已然蜕变从一位只懂书本单文件的“语法的背默初学者”，成长为了一位初探具有无尽能力组合威力的“初级现代 C++ 软件系统架构工程师”了。**

去吧！开启你在深不可测、但充满极致性能挑战的 C++ 世界中的华丽冒险篇章！