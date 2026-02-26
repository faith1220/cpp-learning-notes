# 15.4 std::filesystem

这可能是全世界 C++ 开发者期盼最久、呼声最高的特性。

在 C++17 之前，标准库居然一直没有提供一套能够操作系统目录、文件查询的官方 API。导致任何和文件沾边的稍微高级一点的操作（比如：遍历一个文件夹里的所有 `.txt` 文件，判断某个文件存不存在），开发者都立刻会被迫分裂面临艰难抉择：

1. 要么去调用底层系统相关的黑魔法（Windows 下用 `FindFirstFile` 大魔王，Linux 下用 `dirent.h` POSIX 接口）。一写就失去了可跨平台移植性。
2. 要么去下载引入庞大的第三方 `Boost::filesystem`。

终于，C++17 照搬收编了 Boost 中的文件系统库，将其正式确立为官方标准：`std::filesystem`！

## 核心组件与使用

使用时需要包含头文件 `<filesystem>`。为了少打字，日常开发中常将其命名空间起一个短别名。

```cpp
#include <iostream>
#include <filesystem>
#include <string>

// 起别名是普遍惯例
namespace fs = std::filesystem;

int main() {
    // ---------------------------------------------------------------- //
    // 1. 核心类 fs::path：彻底屏蔽 Windows 反斜杠 \ 和 Linux 斜杠 / 的路径拼接噩梦！
    fs::path dir_path = "data";
    fs::path file_path = dir_path / "logs" / "report.txt"; // 直接用除号 / 进行跨平台无缝拼接！

    std::cout << "生成的跨平台路径是: " << file_path << "\n";
    std::cout << "提取后缀扩展名: " << file_path.extension() << "\n";
    std::cout << "提取纯文件名: " << file_path.filename() << "\n";

    // ---------------------------------------------------------------- //
    // 2. 常规盘问与探测：替代了以前靠尝试打开文件来猜存在的反模式！
    if (!fs::exists(dir_path)) {
        // 创建整个嵌套路径的所有文件夹系列 (相当于 linux 的 mkdir -p)
        fs::create_directories(dir_path / "logs");
        std::cout << "目录不存在，已为您崭新创建！\n";
    }

    // 判明物理正身
    if (fs::is_directory(dir_path)) {
        std::cout << "这是一个有效文件夹。\n";
    }

    // ---------------------------------------------------------------- //
    // 3. 神级功能遍历目录：目录迭代器 (Directory Iterator)
    // 假设当前我们项目底下的某个目录里有东西：
    std::cout << "\n开始遍历当前运行项目所在目录下的文件：\n";

    // fs::directory_iterator 提供浅层遍历
    // fs::recursive_directory_iterator 提供无限深入子文件夹的递归遍历套件！
    for (const auto& entry : fs::directory_iterator(".")) {
        auto path = entry.path();
        if (entry.is_regular_file()) {
            std::cout << "[文件] " << path.filename() << " : "
                      << fs::file_size(path) << " bytes\n";
        } else if (entry.is_directory()) {
            std::cout << "[目录] " << path.filename() << "\n";
        }
    }

    // 4. 便捷的复制删除操作
    // fs::copy(file_path, "backup.txt");
    // fs::remove_all("data/logs"); // 直接删库跑路级指令（整个目录连根拔起）
}
```

有了 `std::filesystem`，C++ 终于在文件操作层面具备了与 Python、Java 这种脚本及上层语言一较高下的、清爽极速的工程实战体验。