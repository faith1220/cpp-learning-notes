# 5.2 C 风格字符串

## 什么是 C 风格字符串

C++ 从 C 语言继承了一种原始的字符串表示方式：以**空字符** `'\0'`（ASCII 值为 0）结尾的字符数组。这种字符串被称为 **C 风格字符串**（C-style String）或**空终止字符串**（Null-terminated String）。

```cpp
char greeting[] = "Hello";
```

上面这行代码实际上创建了一个包含 6 个字符的数组：`'H'`、`'e'`、`'l'`、`'l'`、`'o'`、`'\0'`。末尾的空字符由编译器自动添加，用于标记字符串的结束位置。

也可以逐字符初始化，但必须手动添加空字符：

```cpp
char greeting[] = {'H', 'e', 'l', 'l', 'o', '\0'};   // 等价于 "Hello"
char bad[] = {'H', 'e', 'l', 'l', 'o'};                // 不是合法的 C 风格字符串，缺少 '\0'
```

## 字符串字面量

用双引号包裹的文本（如 `"Hello"`）是**字符串字面量**（String Literal），它的类型是 `const char[]`。字符串字面量存储在只读内存区域，不应被修改：

```cpp
const char* msg = "Hello";   // 指向字符串字面量
// msg[0] = 'h';             // 未定义行为：不能修改字符串字面量
```

## 常用操作函数

C 标准库（通过 `<cstring>` 头文件引入）提供了一组操作 C 风格字符串的函数：

| 函数 | 作用 | 示例 |
|------|------|------|
| `strlen(s)` | 返回字符串长度（不含 `'\0'`） | `strlen("Hello")` → `5` |
| `strcpy(dest, src)` | 将 src 复制到 dest | `strcpy(buf, "Hi")` |
| `strcat(dest, src)` | 将 src 追加到 dest 末尾 | `strcat(buf, " World")` |
| `strcmp(s1, s2)` | 比较两个字符串 | 相等返回 0，s1 < s2 返回负值 |

```cpp
#include <cstring>
#include <iostream>

int main() {
    char name[50];
    strcpy(name, "Zhang");
    strcat(name, " San");
    std::cout << name << std::endl;              // 输出：Zhang San
    std::cout << "长度：" << strlen(name) << std::endl;  // 输出：长度：9

    if (strcmp(name, "Zhang San") == 0) {
        std::cout << "字符串相等" << std::endl;
    }
    return 0;
}
```

## C 风格字符串的风险

C 风格字符串是许多安全漏洞的根源：

- **缓冲区溢出** — `strcpy` 和 `strcat` 不检查目标数组是否有足够空间。如果源字符串超出目标数组的容量，会写入相邻的内存区域，可能导致程序崩溃或被恶意利用。
- **忘记空字符** — 如果字符数组末尾没有 `'\0'`，`strlen`、`std::cout` 等操作会一直读取内存直到偶然遇到零值，产生不可预测的结果。
- **手动管理容量** — 拼接、截取等操作都需要程序员自行计算和分配足够的空间。

{% hint style="danger" %}
在现代 C++ 中，应优先使用 `std::string` 来处理字符串。它自动管理内存、支持直观的操作符、不存在缓冲区溢出风险。C 风格字符串仅在与 C 代码交互或对性能有极端要求的底层场景中使用。
{% endhint %}
