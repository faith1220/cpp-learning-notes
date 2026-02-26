# 4.3 函数重载

## 什么是函数重载

**函数重载**（Function Overloading）是指在同一个作用域内定义多个**同名但参数列表不同**的函数。编译器根据调用时传入的参数类型和数量来决定调用哪个版本。

```cpp
#include <iostream>

int add(int a, int b) {
    return a + b;
}

double add(double a, double b) {
    return a + b;
}

int add(int a, int b, int c) {
    return a + b + c;
}

int main() {
    std::cout << add(3, 5) << std::endl;        // 调用 int add(int, int)，输出 8
    std::cout << add(1.5, 2.3) << std::endl;    // 调用 double add(double, double)，输出 3.8
    std::cout << add(1, 2, 3) << std::endl;     // 调用 int add(int, int, int)，输出 6
    return 0;
}
```

## 重载的规则

编译器区分重载函数依据的是**函数签名**（Function Signature），包括函数名和参数列表（参数的数量、类型、顺序）。

以下情况构成合法的重载：

```cpp
void print(int x);           // 参数类型不同
void print(double x);        //
void print(int x, int y);    // 参数数量不同
void print(double x, int y); // 参数顺序不同
void print(int x, double y); //
```

以下情况**不构成**重载：

```cpp
int compute(int x);      // 仅返回类型不同，不构成重载
double compute(int x);   // 编译错误：与上面的函数冲突

void process(int x);          // 参数名不同不构成重载
void process(int number);     // 编译错误：与上面完全相同
```

关键点：**返回类型不参与重载决策**。两个函数如果只有返回类型不同，编译器无法区分它们。

## 重载决议

当调用一个重载函数时，编译器执行**重载决议**（Overload Resolution）来确定最佳匹配。匹配过程按优先级从高到低：

1. **精确匹配** — 实参类型与形参类型完全一致
2. **类型提升** — 如 `char` → `int`、`float` → `double`
3. **标准转换** — 如 `int` → `double`、`double` → `int`

```cpp
void show(int x)    { std::cout << "int: " << x << std::endl; }
void show(double x) { std::cout << "double: " << x << std::endl; }

int main() {
    show(42);       // 精确匹配 int 版本
    show(3.14);     // 精确匹配 double 版本
    show('A');      // char 提升为 int，调用 int 版本
    show(3.14f);    // float 提升为 double，调用 double 版本
    return 0;
}
```

### 二义性错误

如果编译器无法确定唯一的最佳匹配，会报告**二义性**（Ambiguity）错误：

```cpp
void func(int x, double y);
void func(double x, int y);

int main() {
    func(1, 2);   // 编译错误：两个版本都需要一次转换，无法判断哪个更好
    return 0;
}
```

遇到二义性时，可以通过显式类型转换来消除：

```cpp
func(1, 2.0);   // 明确第二个参数是 double，匹配第一个版本
```

## 重载与默认参数

重载函数与默认参数组合时要小心，避免产生调用歧义：

```cpp
void display(int x);
void display(int x, int y = 10);

int main() {
    display(5);   // 编译错误：两个版本都可以接受一个 int 参数
    return 0;
}
```

设计函数接口时，应避免这种重载与默认参数冲突的情况。
