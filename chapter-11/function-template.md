# 11.1 函数模板

如果你需要写一个交换两个变量值的函数 `swap`，使用非模板的方式，你可能需要使用函数重载来支持不同的类型：

```cpp
void swap(int& a, int& b) { int temp = a; a = b; b = temp; }
void swap(double& a, double& b) { double temp = a; a = b; b = temp; }
void swap(std::string& a, std::string& b) { std::string temp = a; a = b; b = temp; }
// ... 写无数个 ...
```

这显然违反了 DRY（Don't Repeat Yourself）原则。**函数模板**允许我们编写一段通用的逻辑，而将类型作为参数传递。

## 定义函数模板

使用 `template` 关键字和尖括号 `< >` 定义模板参数列表。

```cpp
#include <iostream>
#include <string>

// 模板定义：T 是一个类型占位符
template <typename T>
void my_swap(T& a, T& b) {
    T temp = a;
    a = b;
    b = temp;
}

int main() {
    int x = 10, y = 20;
    my_swap(x, y);       // 编译器自动推导出 T 是 int
    std::cout << "x: " << x << ", y: " << y << "\n";

    std::string s1 = "Hello", s2 = "World";
    my_swap(s1, s2);     // 编译器自动推导出 T 是 std::string
    std::cout << "s1: " << s1 << ", s2: " << s2 << "\n";

    return 0;
}
```

在上面的代码中：
- `template <typename T>` 告诉编译器，接下来的函数是一个模板，`T` 是一种暂未指定的类型。
- （你也可以使用 `class` 关键字代替 `typename`，即 `template <class T>`，它们在模板参数列表这里的作用完全相同。但现代 C++ 推荐使用 `typename`）。

## 模板实例化参数推导

在调用函数模板时，编译器会根据传入的实际参数类型，自动分析（Type Deduction）出 `T` 是什么，并且背后偷偷生成对应类型的函数代码。这个过程叫做**模板的实例化（Instantiation）**。

```cpp
template <typename T>
T add(T a, T b) { return a + b; }

// 编译器偷偷生成了：int add(int, int)
int res1 = add(5, 10);

// 编译器偷偷生成了：double add(double, double)
double res2 = add(3.14, 2.71);
```

### 隐式实例化 vs 显式指定类型

如果函数的参数无法让编译器推导出唯一的类型（例如类型冲突），编译器会报错。此时可以使用**尖括号手工指定类型**参数：

```cpp
int a = 5;
double b = 6.5;

// add(a, b);         // 错误：编译器不知道 T 应该是 int 还是 double！
int r1 = add<int>(a, b);    // 显式告诉编译器 T 是 int (b 会发生隐式转换转成 int)
double r2 = add<double>(a, b); // 显式告诉编译器 T 是 double (a 会转成 double)
```

## 多模板参数

一个函数模板当然可以有多个模板参数，或者包含非类型模板参数。

```cpp
// 两个不同的类型占位符
template <typename T1, typename T2>
void printDouble(T1 a, T2 b) {
    std::cout << a << " and " << b << "\n";
}

printDouble(42, "string"); // T1 被推导为 int, T2 被推导为 const char*
```

## C++14 中的 auto 与 Lambda 模板

在 C++14 之后，你可以使用 `auto` 关键字将普通的函数简写为模板函数（特别是用于 Lambda 表达式）：

```cpp
// 泛型 Lambda (C++14) - 本质上就是个模板函数
auto generic_add = [](auto a, auto b) { return a + b; };

// C++20 允许你在普通函数参数里直接使用 auto，也是函数模板的简写：
// void printVal(auto v) { std::cout << v; }
```