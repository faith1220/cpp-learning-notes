# 6.1 类的定义

## 从结构体说起

在学习类之前，先了解 C++ 的**结构体**（Struct）。结构体可以将多个不同类型的变量组合在一起：

```cpp
struct Student {
    std::string name;
    int age;
    double gpa;
};

int main() {
    Student s;
    s.name = "张三";
    s.age = 20;
    s.gpa = 3.8;
    std::cout << s.name << "，" << s.age << " 岁" << std::endl;
    return 0;
}
```

结构体解决了「将相关数据组合在一起」的问题，但数据和操作数据的函数仍然是分离的。**类**在此基础上更进一步：将数据和行为封装为一个整体。

## 定义一个类

类的定义使用 `class` 关键字：

```cpp
class Rectangle {
public:
    double width;
    double height;

    double area() {
        return width * height;
    }

    double perimeter() {
        return 2 * (width + height);
    }
};   // 注意末尾的分号
```

这个 `Rectangle` 类包含：

- **成员变量**（Member Variable）：`width` 和 `height`，描述矩形的属性
- **成员函数**（Member Function）：`area()` 和 `perimeter()`，定义矩形的行为

成员变量也称为**数据成员**（Data Member）或**字段**（Field），成员函数也称为**方法**（Method）。

## 创建对象

类是一个**蓝图**，对象是根据蓝图创建的**实例**。创建对象的方式与声明变量类似：

```cpp
int main() {
    Rectangle r1;           // 创建一个 Rectangle 对象
    r1.width = 5.0;
    r1.height = 3.0;

    std::cout << "面积：" << r1.area() << std::endl;        // 15
    std::cout << "周长：" << r1.perimeter() << std::endl;    // 16

    Rectangle r2;           // 可以创建多个对象，互不影响
    r2.width = 10.0;
    r2.height = 4.0;
    std::cout << "面积：" << r2.area() << std::endl;        // 40

    return 0;
}
```

通过 `.`（成员访问运算符）访问对象的成员变量和成员函数。每个对象拥有自己独立的成员变量副本，`r1.width` 和 `r2.width` 是不同的变量。

## 在类外定义成员函数

当成员函数的实现较长时，可以只在类内声明，在类外提供定义。类外定义时需要使用**作用域解析运算符** `::`：

```cpp
class Rectangle {
public:
    double width;
    double height;

    double area();           // 类内声明
    double perimeter();      // 类内声明
};

// 类外定义
double Rectangle::area() {
    return width * height;
}

double Rectangle::perimeter() {
    return 2 * (width + height);
}
```

`Rectangle::area()` 表示「属于 `Rectangle` 类的 `area` 函数」。这种分离方式在实际项目中很常见：类的声明放在头文件中，成员函数的定义放在源文件中。

## class 与 struct 的区别

在 C++ 中，`class` 和 `struct` 几乎完全相同，唯一的区别在于**默认访问权限**：

- `struct` 的成员默认是 `public`（公开）
- `class` 的成员默认是 `private`（私有）

```cpp
struct A {
    int x;     // 默认 public
};

class B {
    int x;     // 默认 private
};
```

习惯上，用 `struct` 表示纯数据的简单聚合体（如坐标点、配置项），用 `class` 表示具有行为和封装的类型。这只是一种约定，语法上两者可以互换。

## 一个更完整的示例

```cpp
#include <iostream>
#include <string>

class BankAccount {
public:
    std::string owner;
    double balance;

    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            std::cout << "存入 " << amount << " 元" << std::endl;
        }
    }

    void withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            std::cout << "取出 " << amount << " 元" << std::endl;
        } else {
            std::cout << "余额不足" << std::endl;
        }
    }

    void showBalance() {
        std::cout << owner << " 的余额：" << balance << " 元" << std::endl;
    }
};

int main() {
    BankAccount acc;
    acc.owner = "李四";
    acc.balance = 1000.0;

    acc.showBalance();       // 李四 的余额：1000 元
    acc.deposit(500);        // 存入 500 元
    acc.withdraw(200);       // 取出 200 元
    acc.showBalance();       // 李四 的余额：1300 元

    return 0;
}
```

这个示例演示了类的基本思想：将账户数据（`owner`、`balance`）和操作（`deposit`、`withdraw`、`showBalance`）封装在一起。但它还有一个问题——外部代码可以直接修改 `balance`（如 `acc.balance = -999;`），绕过了存取逻辑。这就引出了下一节要讨论的**访问控制**和构造函数的必要性。
