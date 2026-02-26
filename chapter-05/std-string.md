# 5.3 std::string

## 概述

`std::string` 是 C++ 标准库提供的字符串类，定义在 `<string>` 头文件中。相比 C 风格字符串，它自动管理内存、支持丰富的操作接口，是 C++ 中处理文本的首选方式。

```cpp
#include <string>
#include <iostream>

int main() {
    std::string greeting = "Hello, World!";
    std::cout << greeting << std::endl;
    return 0;
}
```

## 创建与初始化

```cpp
std::string s1;                  // 空字符串 ""
std::string s2 = "Hello";       // 从字符串字面量初始化
std::string s3("Hello");        // 等价写法
std::string s4(5, 'A');         // "AAAAA"（5 个 'A'）
std::string s5 = s2;            // 从另一个 string 复制
std::string s6 = s2 + ", World";  // 拼接初始化
```

## 基本操作

### 获取长度

```cpp
std::string s = "Hello";
std::cout << s.size() << std::endl;     // 5
std::cout << s.length() << std::endl;   // 5（与 size() 完全等价）
std::cout << s.empty() << std::endl;    // false（0）
```

### 访问字符

```cpp
std::string s = "Hello";
std::cout << s[0] << std::endl;     // 'H'（不做越界检查）
std::cout << s.at(1) << std::endl;  // 'e'（越界时抛出 std::out_of_range 异常）
std::cout << s.front() << std::endl; // 'H'（第一个字符）
std::cout << s.back() << std::endl;  // 'o'（最后一个字符）
```

`[]` 和 `.at()` 的区别在于越界行为：`[]` 不检查边界（与原生数组一致），`.at()` 会在越界时抛出异常。

### 拼接

`std::string` 支持用 `+` 和 `+=` 拼接字符串：

```cpp
std::string first = "Hello";
std::string second = ", World!";
std::string result = first + second;     // "Hello, World!"

first += second;                          // first 变为 "Hello, World!"
first += '!';                             // 也可以追加单个字符
```

### 比较

可以直接使用关系运算符比较字符串，按**字典序**（Lexicographic Order）逐字符比较：

```cpp
std::string a = "apple";
std::string b = "banana";

if (a < b) {
    std::cout << a << " 在 " << b << " 之前" << std::endl;
}

if (a == "apple") {
    std::cout << "是 apple" << std::endl;
}
```

## 常用方法

### 查找

```cpp
std::string s = "Hello, World!";

// find：返回子串首次出现的位置，未找到返回 std::string::npos
size_t pos = s.find("World");
if (pos != std::string::npos) {
    std::cout << "找到，位置：" << pos << std::endl;   // 位置：7
}

// rfind：从后往前查找
size_t rpos = s.rfind('l');   // 最后一个 'l' 的位置：10
```

### 截取子串

```cpp
std::string s = "Hello, World!";
std::string sub = s.substr(7, 5);   // 从位置 7 开始，取 5 个字符
std::cout << sub << std::endl;       // "World"
```

### 插入与删除

```cpp
std::string s = "Hello World";
s.insert(5, ",");          // 在位置 5 插入逗号 → "Hello, World"
s.erase(5, 1);             // 从位置 5 开始删除 1 个字符 → "Hello World"
```

### 替换

```cpp
std::string s = "Hello, World!";
s.replace(7, 5, "C++");    // 从位置 7 开始，替换 5 个字符为 "C++"
std::cout << s << std::endl; // "Hello, C++!"
```

## 遍历

```cpp
std::string s = "Hello";

// 下标遍历
for (size_t i = 0; i < s.size(); ++i) {
    std::cout << s[i];
}

// 范围 for 循环（推荐）
for (char c : s) {
    std::cout << c;
}

// 修改每个字符
for (char& c : s) {
    c = std::toupper(c);   // 转为大写（需要 <cctype>）
}
```

## 与 C 风格字符串的转换

```cpp
std::string cppStr = "Hello";

// std::string → C 风格字符串
const char* cStr = cppStr.c_str();   // 返回以 '\0' 结尾的 const char*

// C 风格字符串 → std::string
const char* raw = "World";
std::string fromC = raw;             // 隐式转换
std::string fromC2(raw);             // 显式构造
```

`c_str()` 返回的指针指向 `std::string` 内部的缓冲区，当 `std::string` 被修改或销毁后，该指针会失效。

## 数值与字符串的转换

C++11 提供了数值和字符串之间的转换函数：

```cpp
#include <string>

// 数值 → 字符串
std::string s1 = std::to_string(42);       // "42"
std::string s2 = std::to_string(3.14);     // "3.140000"

// 字符串 → 数值
int n = std::stoi("42");          // 42
double d = std::stod("3.14");     // 3.14
long l = std::stol("1000000");    // 1000000
```

如果字符串无法转换为目标类型，这些函数会抛出 `std::invalid_argument` 异常。
