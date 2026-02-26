# 2.4 类型转换

在程序中，不同类型的数据经常需要混合运算或相互赋值。C++ 提供了**隐式转换**和**显式转换**两种机制来处理类型之间的差异。

## 隐式转换

**隐式转换**（Implicit Conversion）是编译器在需要时自动执行的类型转换，无需程序员显式指定。

### 算术转换

当不同类型的数值参与运算时，编译器会将较「窄」的类型自动提升为较「宽」的类型，以避免数据丢失。提升方向大致如下：

```
bool → char → short → int → long → long long → float → double → long double
```

```cpp
int a = 10;
double b = 3.5;
auto result = a + b;   // a 被隐式转换为 double，result 的类型是 double
```

### 赋值转换

将一种类型的值赋给另一种类型的变量时，值会被转换为目标变量的类型：

```cpp
int x = 3.14;      // double → int，小数部分被截断，x 为 3
double y = 42;     // int → double，y 为 42.0
bool flag = -1;    // int → bool，非零值转为 true
int n = true;      // bool → int，n 为 1
```

{% hint style="warning" %}
将较大范围的类型赋值给较小范围的类型时，可能发生数据丢失。这种转换称为**窄化转换**（Narrowing Conversion）。编译器可能会发出警告，但不一定报错。使用花括号初始化（`int x{3.14};`）可以让编译器将窄化转换视为错误。
{% endhint %}

### 整数提升

在运算中，`bool`、`char`、`short` 等小于 `int` 的类型会自动提升为 `int`：

```cpp
char a = 'A';       // 65
char b = 'B';       // 66
auto sum = a + b;   // a 和 b 先提升为 int，sum 的类型是 int，值为 131
```

## 显式转换

当你明确需要进行类型转换时，应使用**显式转换**（Explicit Conversion），也称为**强制类型转换**（Cast）。C++ 提供了四种命名转换运算符，以及从 C 语言继承的旧式转换语法。

### static_cast（推荐）

`static_cast` 是最常用的转换方式，用于在相关类型之间进行编译期转换：

```cpp
double pi = 3.14159;
int intPi = static_cast<int>(pi);   // 显式截断为 3

int a = 5, b = 3;
double ratio = static_cast<double>(a) / b;  // 先将 a 转为 double，再做浮点除法
// ratio 为 1.666...
```

`static_cast` 的优点在于意图清晰、编译器会检查转换是否合理。如果转换在语义上不合法（例如将指针转换为不相关的类型），编译器会报错。

### C 风格转换（不推荐）

C 语言的转换语法在 C++ 中仍然可用，但不推荐使用：

```cpp
int x = (int)3.14;         // C 风格
int y = int(3.14);         // 函数风格（也是 C 风格的变体）
```

这种写法的问题在于它不区分转换的类型和意图，难以在代码中搜索，且可能执行危险的转换而不报错。在现代 C++ 中，应始终使用命名转换运算符。

### 其他转换运算符

C++ 还提供了另外三种转换运算符，用于特定场景：

- **`const_cast`** — 添加或移除 `const` 限定符。用于极少数需要修改常量引用的场景。
- **`reinterpret_cast`** — 执行低级别的位模式重新解释，几乎不做任何检查。仅用于底层编程。
- **`dynamic_cast`** — 用于含有虚函数的类层次结构中的安全向下转型。会在运行时检查转换是否有效。

这三种转换运算符涉及指针、继承等进阶主题，将在后续相关章节中介绍。初学阶段只需掌握 `static_cast` 即可。

## 实际应用示例

以下程序演示了类型转换的典型用途——整数除法转为浮点除法：

```cpp
#include <iostream>

int main() {
    int total = 17;
    int count = 5;

    // 整数除法：结果为 3
    std::cout << "整数除法：" << total / count << std::endl;

    // 使用 static_cast 转为浮点除法：结果为 3.4
    double average = static_cast<double>(total) / count;
    std::cout << "浮点除法：" << average << std::endl;

    return 0;
}
```

输出：

```
整数除法：3
浮点除法：3.4
```

这是日常编程中最常见的显式转换场景之一。当你看到整数除法的结果不符合预期时，首先检查是否需要将操作数转换为浮点类型。
