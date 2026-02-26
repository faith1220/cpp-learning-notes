# 8.3 类型转换运算符

## 两个方向的转换

自定义类型与其他类型之间的转换有两个方向：

- **其他类型 → 自定义类型**：通过**转换构造函数**实现
- **自定义类型 → 其他类型**：通过**类型转换运算符**实现

## 转换构造函数

接受**单个参数**的构造函数可以充当从参数类型到类类型的隐式转换：

```cpp
class Fraction {
private:
    int num, den;

public:
    // 转换构造函数：int → Fraction
    Fraction(int n) : num(n), den(1) {}

    Fraction(int n, int d) : num(n), den(d) {}

    friend std::ostream& operator<<(std::ostream& os, const Fraction& f) {
        return os << f.num << "/" << f.den;
    }
};

void print(const Fraction& f) {
    std::cout << f << std::endl;
}

int main() {
    Fraction f1 = 5;      // 隐式转换：int → Fraction(5, 1)
    Fraction f2(3, 4);

    print(7);              // 隐式转换：7 → Fraction(7)
    return 0;
}
```

`Fraction(int n)` 既是构造函数，也充当了从 `int` 到 `Fraction` 的隐式转换规则。

### explicit 阻止隐式转换

隐式转换有时会导致意外的行为。使用 `explicit` 关键字可以禁止隐式转换，只允许显式构造：

```cpp
class Fraction {
private:
    int num, den;

public:
    explicit Fraction(int n) : num(n), den(1) {}
    Fraction(int n, int d) : num(n), den(d) {}
};

int main() {
    // Fraction f1 = 5;       // 编译错误！explicit 禁止隐式转换
    Fraction f2(5);            // 正确：显式构造
    Fraction f3 = Fraction(5); // 正确：显式构造

    // print(7);              // 编译错误！
    print(Fraction(7));        // 正确：显式构造后传入
    return 0;
}
```

{% hint style="warning" %}
经验法则：单参数构造函数（或所有参数都有默认值的构造函数）通常应该声明为 `explicit`，除非隐式转换确实是你想要的语义。这能避免很多难以察觉的错误。
{% endhint %}

## 类型转换运算符

类型转换运算符实现从自定义类型到其他类型的转换。语法是 `operator 目标类型()`，没有返回类型声明（返回类型就是运算符名称中的目标类型）：

```cpp
class Fraction {
private:
    int num, den;

public:
    Fraction(int n, int d) : num(n), den(d) {}

    // 类型转换运算符：Fraction → double
    operator double() const {
        return static_cast<double>(num) / den;
    }
};

int main() {
    Fraction f(3, 4);
    double val = f;           // 隐式转换：调用 operator double()
    std::cout << val << std::endl;   // 0.75

    // 也可以用在表达式中
    double result = f + 1.5;  // f 先转为 double(0.75)，再加 1.5
    std::cout << result << std::endl; // 2.25

    return 0;
}
```

## explicit 类型转换运算符（C++11）

与转换构造函数类似，类型转换运算符也可以声明为 `explicit`，只允许显式转换：

```cpp
class Fraction {
private:
    int num, den;

public:
    Fraction(int n, int d) : num(n), den(d) {}

    explicit operator double() const {
        return static_cast<double>(num) / den;
    }
};

int main() {
    Fraction f(3, 4);

    // double val = f;              // 编译错误！explicit 禁止隐式转换
    double val = static_cast<double>(f);  // 正确：显式转换
    double val2 = (double)f;              // 正确：C 风格强制转换（不推荐）

    return 0;
}
```

### 特例：operator bool()

`explicit operator bool()` 有一个特殊待遇：在**条件上下文**中（`if`、`while`、`for`、`?:`、`&&`、`||`、`!`），即使是 `explicit` 的，也会自动转换。这称为**上下文转换**（Contextual Conversion）：

```cpp
class Connection {
private:
    bool connected;

public:
    Connection(bool c) : connected(c) {}

    explicit operator bool() const {
        return connected;
    }
};

int main() {
    Connection conn(true);

    // bool b = conn;            // 编译错误！explicit 阻止隐式转换
    if (conn) {                   // 正确！条件上下文允许 explicit bool
        std::cout << "已连接" << std::endl;
    }

    // int x = conn;             // 编译错误！不会意外转为 int
    // Connection c2 = conn + 1; // 编译错误！不会参与算术运算

    return 0;
}
```

这正是 `explicit operator bool()` 的设计意图：允许在 `if` 中自然地检查状态，同时防止对象意外地参与算术运算或赋值给整型变量。标准库中 `std::ifstream`、`std::optional` 等类型都使用了这个模式。

## 隐式转换的陷阱

当一个类同时有转换构造函数和类型转换运算符时，可能导致二义性：

```cpp
class A {
public:
    A(int x) {}               // int → A
    operator int() const { return 0; }  // A → int
};

class B {
public:
    B(const A& a) {}          // A → B
};

void foo(B b) {}
void foo(int n) {}

int main() {
    A a(1);
    // foo(a);   // 二义性！a 可以转为 int 调用 foo(int)，
                 //          也可以转为 B 调用 foo(B)
    return 0;
}
```

避免此类问题的原则：

1. 尽量使用 `explicit` 限制隐式转换
2. 一个类最多提供一个非 explicit 的类型转换运算符
3. 避免同时定义「到算术类型的转换」和「接受算术类型的构造函数」

## 完整示例：安全的布尔转换

```cpp
#include <iostream>
#include <string>

class OptionalInt {
private:
    int value;
    bool hasValue;

public:
    OptionalInt() : value(0), hasValue(false) {}

    explicit OptionalInt(int v) : value(v), hasValue(true) {}

    // 安全的布尔转换：检查是否有值
    explicit operator bool() const {
        return hasValue;
    }

    int get() const {
        if (!hasValue) {
            throw std::runtime_error("没有值");
        }
        return value;
    }

    friend std::ostream& operator<<(std::ostream& os, const OptionalInt& opt) {
        if (opt.hasValue) {
            return os << opt.value;
        }
        return os << "(空)";
    }
};

int main() {
    OptionalInt a;
    OptionalInt b(42);

    if (a) {
        std::cout << "a = " << a.get() << std::endl;
    } else {
        std::cout << "a 没有值" << std::endl;    // 输出这行
    }

    if (b) {
        std::cout << "b = " << b.get() << std::endl;  // b = 42
    }

    // 以下都会被 explicit 阻止：
    // int x = b;         // 错误
    // bool y = b;        // 错误
    // OptionalInt c = 5; // 错误

    return 0;
}
```

## 总结

| 转换方向 | 实现方式 | 建议 |
|---------|---------|------|
| 其他类型 → 自定义类型 | 转换构造函数 | 默认加 `explicit` |
| 自定义类型 → 其他类型 | `operator 类型()` | 默认加 `explicit` |
| 自定义类型 → bool | `explicit operator bool()` | 几乎总是用 explicit |

核心原则：**默认使用 `explicit`**，只在隐式转换确实有意义且不会引起歧义时才去掉它。
