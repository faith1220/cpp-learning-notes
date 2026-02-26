# 2.3 输入与输出

在第一章中我们已经使用 `std::cout` 输出了 "Hello, World!"。本节将系统介绍 C++ 的标准输入输出机制，包括如何从键盘读取数据以及如何格式化输出。

## 标准输出：std::cout

`std::cout`（Console Output）配合插入运算符 `<<` 将数据输出到终端：

```cpp
#include <iostream>

int main() {
    int age = 20;
    double height = 1.75;

    std::cout << "年龄：" << age << std::endl;
    std::cout << "身高：" << height << " 米" << std::endl;
    return 0;
}
```

输出：

```
年龄：20
身高：1.75 米
```

几个要点：

- 多个 `<<` 可以**链式调用**，将不同类型的数据依次送入输出流
- `std::endl` 输出换行符并**刷新缓冲区**（Flush Buffer）
- 也可以用 `'\n'` 来换行，它不会刷新缓冲区，性能略优于 `std::endl`

```cpp
std::cout << "第一行\n";       // 用 \n 换行
std::cout << "第二行" << std::endl;  // 用 endl 换行并刷新
```

在大多数场景下，两者的差异可以忽略。需要确保输出立即显示时（如调试信息），使用 `std::endl`；追求输出性能时（如大量数据输出），使用 `'\n'`。

## 标准输入：std::cin

`std::cin`（Console Input）配合提取运算符 `>>` 从键盘读取数据：

```cpp
#include <iostream>

int main() {
    int age;
    std::cout << "请输入你的年龄：";
    std::cin >> age;
    std::cout << "你的年龄是 " << age << " 岁" << std::endl;
    return 0;
}
```

运行效果：

```
请输入你的年龄：20
你的年龄是 20 岁
```

`std::cin` 会自动根据变量的类型来解析输入。读取整数时会跳过前导空白字符（空格、换行、制表符），遇到非数字字符时停止。

### 读取多个值

可以在一条语句中连续读取多个变量：

```cpp
int x, y;
std::cout << "输入两个整数（用空格分隔）：";
std::cin >> x >> y;
std::cout << "它们的和是 " << x + y << std::endl;
```

用户输入 `3 5` 后，`x` 得到 `3`，`y` 得到 `5`。`std::cin` 默认以空白字符作为分隔。

### 读取字符串

使用 `std::cin >> variable` 读取字符串时，它会在遇到空白字符时停止，因此只能读取单个单词：

```cpp
#include <iostream>
#include <string>

int main() {
    std::string name;
    std::cout << "你的名字：";
    std::cin >> name;    // 如果输入 "Zhang San"，只会读取 "Zhang"
    std::cout << "你好，" << name << std::endl;
    return 0;
}
```

要读取包含空格的整行输入，使用 `std::getline`：

```cpp
std::string fullName;
std::cout << "你的全名：";
std::getline(std::cin, fullName);    // 读取整行，直到遇到换行符
std::cout << "你好，" << fullName << std::endl;
```

### cin 与 getline 混用的陷阱

当 `std::cin >>` 和 `std::getline` 混合使用时，可能遇到一个常见问题：

```cpp
int age;
std::string name;

std::cout << "年龄：";
std::cin >> age;              // 读取数字后，换行符留在缓冲区

std::cout << "姓名：";
std::getline(std::cin, name); // 直接读到了残留的换行符，name 为空
```

解决方法是在 `std::getline` 之前调用 `std::cin.ignore()` 清除缓冲区中残留的换行符：

```cpp
std::cin >> age;
std::cin.ignore();              // 忽略缓冲区中的一个字符（换行符）
std::getline(std::cin, name);   // 现在可以正常读取
```

## 输入错误处理

如果用户输入了与期望类型不匹配的数据（例如需要整数时输入了字母），`std::cin` 会进入**失败状态**（Fail State），后续的所有读取操作都会跳过。可以通过检查流状态来处理：

```cpp
int number;
std::cout << "请输入一个整数：";
std::cin >> number;

if (std::cin.fail()) {
    std::cout << "输入无效，不是整数" << std::endl;
    std::cin.clear();              // 清除错误状态
    std::cin.ignore(1000, '\n');   // 丢弃缓冲区中的错误输入
}
```

输入验证是编写健壮程序的重要环节，但在学习初期可以暂时假设用户总是输入正确格式的数据，优先关注语言本身的特性。
