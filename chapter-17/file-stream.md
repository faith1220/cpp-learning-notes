# 17.1 文件流操作

要在 C++ 中对物理硬盘上的文件展开读写，需要包含标准头文件 `<fstream>`（File Stream）。

在这个头文件里，定义了三种最常用的类：
- `std::ifstream`：专门作输入（读取）用的文件流类（Input File Stream）。
- `std::ofstream`：专门作输出（写入）用的文件流类（Output File Stream）。
- `std::fstream`：读写全能的文件流类。

## 写入文本文件

往文件里写字，和往屏幕上打字没什么两样，都是使用插入流运算符 `<<`：

```cpp
#include <iostream>
#include <fstream>
#include <string>

int main() {
    // 1. 创建 ofstream 对象并尝试打开一个文件用于写入
    std::ofstream outFile("greetings.txt");

    // 2. 判断是否真正在硬盘上打开成功
    // （如果遇到了权限不足，或者磁盘满的情况，就会打不开）
    if (!outFile.is_open()) {
        std::cerr << "文件打开/创建失败！\n";
        return 1;
    }

    // 3. 像用 cout 一样肆意妄为吧！
    outFile << "Hello, File System!\n";
    outFile << "这是一首简单的歌。\n";
    int score = 100;
    outFile << "得分: " << score << "\n";

    // 4. 关闭文件释放内部句柄和锁占用的资源
    // （注：因为 ofstream 符合 RAII 规范，它其实会在离开作用域时通过析构函数自动调用 close()）
    outFile.close();

    std::cout << "文件写入完毕。\n";
    return 0;
}
```

### 追加模式写入

如果我们单纯用 `ofstream outFile("data.txt")` 打开一个已经有数据存在的老文件，**默认行为是把老文件里的内容全部清空删光（Truncate）并重新从头写**！

如果我们仅仅是想把新的日志记录追加到文件的尾巴后面，我们需要同时上报**文件打开模式标志（Open Modes）**：

```cpp
// 引入 std::ios::app (app = append 打死不擦除，就在屁股后面追)
std::ofstream logFile("log.txt", std::ios::app);
logFile << "系统于今日 12:00 例行启动。\n";
```

## 读取文本文件

这涉及读取。按理说提取流运算符 `>>` 就能对付，但是 `>>` 遇到空格或者换行就会自动断开，这使得它读起文章来很不流畅。

我们平时读文件一般分为两种流派：**全自动逐行读取** 或 **一口全吞**。

### 按行读取（最常规稳定）

借助 `<string>` 里面专配的函数 `getline()` 去榨取流。

```cpp
#include <iostream>
#include <fstream>
#include <string>

int main() {
    std::ifstream inFile("greetings.txt");

    if (!inFile) { // 直接如果文件不可用可以直接放 if 里作真伪判定！这也是流对象内部重载了类 bool() 转化的功劳
        std::cerr << "找不到 greetings.txt 呀！\n";
        return 1;
    }

    std::string line;
    // std::getline 会一直从 inFile 肚子里抽丝，直到碰到换行符为止塞给 line
    // 当没东西可抽了（到了文件末尾 EOF），整个函数就会返回假而退潮退出 while！
    while (std::getline(inFile, line)) {
        std::cout << "[文件读出] -> " << line << "\n";
    }

    return 0;
}
```

### 一口吞下整个文件内容

有时候文件很小（比如就一个 json 配置文件），一行行读太费周折了。可以用这段使用现代 C++ 思维的极简“迭代器吞噬法”：

```cpp
#include <fstream>
#include <string>
#include <sstream>

std::string readAll(const std::string& path) {
    std::ifstream file(path);
    if(!file) return ""; // 可以选择抛异常

    // 最简单粗暴吞字大法：借用 stringstream
    std::stringstream buffer;
    buffer << file.rdbuf();

    return buffer.str();
}
```

## 二进制数据流读写

如果你的文件不是拿给人类眼睛看来读的文本（比如游戏存档、一张图片、一个 `.zip` 压缩包）。这就叫二进制文件。

在这类模式下：
1. 打开时一定要在标志里拼装加注：`std::ios::binary`！绝对不能漏！一旦漏了，系统就会偷偷把读取到的换行回车类控制符号强行翻译转码，直接毁掉你的数据（特别是 Windows 下把 `\r\n` 翻译成 `\n`）。
2. 不准用 `<<` 或者 `getline`。**只能用直爽暴力拆装内存的方法：`.read()` 和 `.write()`。**

```cpp
#include <iostream>
#include <fstream>
#include <vector>

int main() {
    // 写入一个数组背后的纯内存结构到二进制文件当中
    std::vector<int> numbers = {10, 20, 30, 40};

    // 打开必须加 std::ios::binary ！！！
    std::ofstream out("data.bin", std::ios::binary);

    // 参数1：要求必须是底层的字符纸质指针(char*)，需要用指针大强转手段 reinterpret_cast
    // 参数2：要暴力写出多少个字节的坑位
    out.write(reinterpret_cast<const char*>(numbers.data()), numbers.size() * sizeof(int));
    out.close();

    // ==========================================

    // 再暴力生吞读回来
    std::vector<int> read_nums(4); // 提前先分好坑位用来接
    std::ifstream in("data.bin", std::ios::binary);
    in.read(reinterpret_cast<char*>(read_nums.data()), 4 * sizeof(int));

    std::cout << "读回来的第二个数字是: " << read_nums[1] << "\n"; // 输出 20

    return 0;
}
```