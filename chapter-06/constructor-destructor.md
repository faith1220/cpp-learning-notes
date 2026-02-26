# 6.2 构造函数与析构函数

## 为什么需要构造函数

上一节的 `BankAccount` 示例中，创建对象后需要逐一赋值：

```cpp
BankAccount acc;
acc.owner = "李四";
acc.balance = 1000.0;
```

这种方式有两个问题：

1. 如果忘记赋值，成员变量将处于**未初始化**状态，`balance` 可能是一个随机数
2. 无法在赋值前对数据进行**合法性检查**（如余额不能为负数）

**构造函数**（Constructor）正是为解决这些问题而设计的：它在对象创建时自动调用，负责完成初始化工作。

## 构造函数的基本语法

构造函数是一种特殊的成员函数，具有以下特征：

- 函数名与类名**完全相同**
- **没有返回类型**（连 `void` 也没有）
- 在对象创建时**自动调用**，不需要手动调用

```cpp
class Rectangle {
public:
    double width;
    double height;

    // 构造函数
    Rectangle(double w, double h) {
        width = w;
        height = h;
    }

    double area() {
        return width * height;
    }
};

int main() {
    Rectangle r(5.0, 3.0);    // 创建对象时传入参数，自动调用构造函数
    std::cout << r.area() << std::endl;  // 15
    return 0;
}
```

创建 `r` 的那一刻，编译器自动调用 `Rectangle(5.0, 3.0)`，将 `width` 设为 5.0、`height` 设为 3.0。

## 成员初始化列表

除了在构造函数体内赋值，C++ 还提供了一种更高效的初始化方式——**成员初始化列表**（Member Initializer List）：

```cpp
class Rectangle {
public:
    double width;
    double height;

    // 使用成员初始化列表
    Rectangle(double w, double h) : width(w), height(h) {
        // 函数体可以为空，也可以放其他逻辑
    }
};
```

初始化列表写在参数列表后面，以 `:` 开头，多个成员之间用 `,` 分隔。

{% hint style="info" %}
成员初始化列表与函数体内赋值的区别：初始化列表是在对象构造时**直接初始化**成员，而函数体内的 `=` 是先默认初始化再**赋值**。对于基本类型差别不大，但对于 `std::string` 等复杂类型，初始化列表避免了一次多余的默认构造，效率更高。推荐优先使用初始化列表。
{% endhint %}

成员的初始化顺序由**声明顺序**决定，与初始化列表中的书写顺序无关：

```cpp
class Example {
public:
    int a;
    int b;

    // 虽然写的是 b(y), a(x)，但 a 仍然先初始化
    Example(int x, int y) : b(y), a(x) {}
};
```

{% hint style="warning" %}
如果成员的初始化依赖其他成员的值，一定要注意声明顺序，否则可能使用到未初始化的值。
{% endhint %}

## 默认构造函数

**默认构造函数**（Default Constructor）是不需要任何参数即可调用的构造函数：

```cpp
class Point {
public:
    double x;
    double y;

    Point() : x(0.0), y(0.0) {}   // 默认构造函数
};

int main() {
    Point p;    // 调用默认构造函数，x=0, y=0
    return 0;
}
```

如果一个类**没有定义任何构造函数**，编译器会自动生成一个默认构造函数（称为**合成默认构造函数**，Synthesized Default Constructor）。但一旦定义了任何构造函数，编译器就不再自动生成：

```cpp
class Rectangle {
public:
    double width;
    double height;

    Rectangle(double w, double h) : width(w), height(h) {}
};

int main() {
    Rectangle r1(5.0, 3.0);   // 正确
    Rectangle r2;              // 错误！没有默认构造函数
    return 0;
}
```

如果希望同时保留带参数和不带参数的创建方式，可以：

**方法一：显式定义默认构造函数**

```cpp
class Rectangle {
public:
    double width;
    double height;

    Rectangle() : width(0.0), height(0.0) {}
    Rectangle(double w, double h) : width(w), height(h) {}
};
```

**方法二：使用默认参数**

```cpp
class Rectangle {
public:
    double width;
    double height;

    Rectangle(double w = 0.0, double h = 0.0) : width(w), height(h) {}
};
```

**方法三：使用 `= default`（C++11）**

```cpp
class Rectangle {
public:
    double width;
    double height;

    Rectangle() = default;     // 让编译器生成默认构造函数
    Rectangle(double w, double h) : width(w), height(h) {}
};
```

`= default` 明确告诉编译器「请帮我生成这个函数」，语义比手写空函数体更清晰。

## 构造函数重载

构造函数可以像普通函数一样进行**重载**，为对象提供多种创建方式：

```cpp
#include <iostream>
#include <string>

class Student {
public:
    std::string name;
    int age;
    double gpa;

    // 无参构造
    Student() : name("未知"), age(0), gpa(0.0) {}

    // 只提供姓名
    Student(const std::string& n) : name(n), age(0), gpa(0.0) {}

    // 提供全部信息
    Student(const std::string& n, int a, double g)
        : name(n), age(a), gpa(g) {}
};

int main() {
    Student s1;                          // 调用无参构造
    Student s2("张三");                  // 调用单参数构造
    Student s3("李四", 20, 3.9);         // 调用三参数构造
    return 0;
}
```

## 委托构造函数（C++11）

当多个构造函数有重复的初始化逻辑时，可以用**委托构造函数**（Delegating Constructor）避免代码重复：

```cpp
class Student {
public:
    std::string name;
    int age;
    double gpa;

    // 主构造函数
    Student(const std::string& n, int a, double g)
        : name(n), age(a), gpa(g) {}

    // 委托给主构造函数
    Student() : Student("未知", 0, 0.0) {}
    Student(const std::string& n) : Student(n, 0, 0.0) {}
};
```

委托构造函数在初始化列表中调用同一个类的其他构造函数，将实际初始化工作集中在一处。

## 析构函数

**析构函数**（Destructor）与构造函数相反，在对象**销毁**时自动调用，负责清理工作。析构函数的特征：

- 函数名是 `~` 加类名
- 没有参数，没有返回类型
- 每个类只能有**一个**析构函数（不能重载）
- 对象离开作用域或被 `delete` 时自动调用

```cpp
#include <iostream>

class Logger {
public:
    std::string tag;

    Logger(const std::string& t) : tag(t) {
        std::cout << "[" << tag << "] 创建" << std::endl;
    }

    ~Logger() {
        std::cout << "[" << tag << "] 销毁" << std::endl;
    }
};

int main() {
    Logger a("A");
    {
        Logger b("B");
        Logger c("C");
    }   // b 和 c 在此处销毁
    Logger d("D");
    return 0;
}   // d 和 a 在此处销毁
```

输出：

```
[A] 创建
[B] 创建
[C] 创建
[C] 销毁
[B] 销毁
[D] 创建
[D] 销毁
[A] 销毁
```

注意销毁顺序与创建顺序**相反**——这是因为局部对象存储在栈上，遵循后进先出（LIFO）原则。

## 析构函数的用途

对于只包含基本类型和标准库类型（如 `std::string`、`std::vector`）的类，通常不需要手写析构函数，编译器生成的默认版本就足够了。析构函数的典型使用场景：

- 释放动态分配的内存（`new` / `delete`）
- 关闭文件句柄
- 释放网络连接等系统资源

```cpp
class DynamicArray {
public:
    int* data;
    int size;

    DynamicArray(int n) : size(n) {
        data = new int[n]();    // 动态分配，() 表示零初始化
    }

    ~DynamicArray() {
        delete[] data;          // 释放内存，避免内存泄漏
        std::cout << "内存已释放" << std::endl;
    }
};
```

{% hint style="warning" %}
这个示例仅为说明析构函数的作用。实际开发中，应优先使用 `std::vector` 而非手动管理内存。第十章将详细讨论资源管理策略。
{% endhint %}

## 构造与析构的完整生命周期

| 阶段 | 发生的事情 |
|------|-----------|
| 分配内存 | 系统为对象分配所需的内存空间 |
| 构造函数 | 初始化成员变量，执行构造函数体 |
| 对象使用 | 调用成员函数，访问成员变量 |
| 析构函数 | 执行清理代码，释放资源 |
| 释放内存 | 系统回收对象占用的内存空间 |

## 常见问题

**Q：构造函数可以手动调用吗？**

一般情况下不直接调用。不过 C++ 提供了**放置 new**（Placement New）等高级机制可以在已有内存上调用构造函数，这属于进阶话题。

**Q：析构函数可以手动调用吗？**

语法上可以（`obj.~ClassName()`），但几乎不应该这样做。手动调用后对象仍在作用域内，离开时会再次析构，导致**双重释放**（Double Free）错误。

**Q：如果构造函数里抛出异常会怎样？**

对象不会被完整构造，析构函数也**不会被调用**。已经构造完成的成员会各自调用自己的析构函数。这是使用 RAII 模式管理资源的重要原因之一，将在第十章详细讨论。
