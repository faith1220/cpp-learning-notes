# 8.2 常见运算符重载

本节以一个 `Vector2D` 类为主线，逐步演示各类运算符的重载方法。

## 算术运算符

### 二元算术运算符（+、-、*、/）

推荐以非成员函数的形式重载，通常基于复合赋值运算符来实现：

```cpp
#include <iostream>

class Vector2D {
public:
    double x, y;

    Vector2D(double x = 0, double y = 0) : x(x), y(y) {}

    // 复合赋值运算符（成员函数）
    Vector2D& operator+=(const Vector2D& rhs) {
        x += rhs.x;
        y += rhs.y;
        return *this;
    }

    Vector2D& operator-=(const Vector2D& rhs) {
        x -= rhs.x;
        y -= rhs.y;
        return *this;
    }
};

// 基于 += 实现 +（非成员函数）
Vector2D operator+(Vector2D lhs, const Vector2D& rhs) {
    lhs += rhs;       // lhs 是值传递的副本，可以直接修改
    return lhs;
}

Vector2D operator-(Vector2D lhs, const Vector2D& rhs) {
    lhs -= rhs;
    return lhs;
}
```

这种写法的好处是：`+` 的逻辑只需要在 `+=` 中维护一份，减少重复代码。注意 `operator+` 的第一个参数是**值传递**——传入时就创建了副本，不会修改原对象。

### 一元运算符（取负）

```cpp
class Vector2D {
public:
    double x, y;

    Vector2D(double x = 0, double y = 0) : x(x), y(y) {}

    // 一元取负
    Vector2D operator-() const {
        return Vector2D(-x, -y);
    }
};

int main() {
    Vector2D v(3, 4);
    Vector2D neg = -v;    // (-3, -4)
    return 0;
}
```

### 标量乘法

向量与标量的乘法需要支持两种写法：`v * 2.0` 和 `2.0 * v`。成员函数只能处理前者，因此需要同时提供非成员函数版本：

```cpp
class Vector2D {
public:
    double x, y;
    Vector2D(double x = 0, double y = 0) : x(x), y(y) {}

    // v * scalar
    Vector2D operator*(double scalar) const {
        return Vector2D(x * scalar, y * scalar);
    }
};

// scalar * v（非成员函数，委托给成员版本）
Vector2D operator*(double scalar, const Vector2D& v) {
    return v * scalar;
}
```

## 比较运算符

### == 和 !=

```cpp
class Vector2D {
public:
    double x, y;
    Vector2D(double x = 0, double y = 0) : x(x), y(y) {}

    bool operator==(const Vector2D& rhs) const {
        return x == rhs.x && y == rhs.y;
    }

    bool operator!=(const Vector2D& rhs) const {
        return !(*this == rhs);    // 基于 == 实现
    }
};
```

{% hint style="info" %}
C++20 引入了 `operator<=>`（三路比较运算符，也称「宇宙飞船运算符」），可以一次性生成所有比较运算符。对于支持 C++20 的项目，这是更简洁的做法，但本书以 C++17 为主，暂不展开。
{% endhint %}

### < 和其他关系运算符

如果类型支持排序，通常先实现 `<`，其余基于它推导：

```cpp
class Fraction {
private:
    int num, den;   // 分子、分母

public:
    Fraction(int n, int d) : num(n), den(d) {}

    bool operator<(const Fraction& rhs) const {
        return num * rhs.den < rhs.num * den;
    }

    bool operator>(const Fraction& rhs) const {
        return rhs < *this;
    }

    bool operator<=(const Fraction& rhs) const {
        return !(rhs < *this);
    }

    bool operator>=(const Fraction& rhs) const {
        return !(*this < rhs);
    }
};
```

## 赋值运算符

### 拷贝赋值运算符（operator=）

当一个已存在的对象被赋予新值时调用：

```cpp
class MyString {
private:
    char* data;
    int length;

public:
    MyString(const char* str = "") {
        length = std::strlen(str);
        data = new char[length + 1];
        std::strcpy(data, str);
    }

    // 拷贝构造函数
    MyString(const MyString& other) {
        length = other.length;
        data = new char[length + 1];
        std::strcpy(data, other.data);
    }

    // 拷贝赋值运算符
    MyString& operator=(const MyString& other) {
        if (this != &other) {           // 防止自赋值
            delete[] data;              // 释放旧内存
            length = other.length;
            data = new char[length + 1];
            std::strcpy(data, other.data);
        }
        return *this;                   // 返回自身引用，支持链式赋值
    }

    ~MyString() {
        delete[] data;
    }
};
```

拷贝赋值运算符的三个要点：

1. **检查自赋值** — `a = a` 时如果先 delete 再拷贝，会访问已释放的内存
2. **释放旧资源** — 赋值前释放当前持有的资源
3. **返回 `*this` 引用** — 支持 `a = b = c` 的链式赋值

{% hint style="warning" %}
如果一个类需要自定义拷贝构造函数、拷贝赋值运算符或析构函数中的任何一个，通常三个都需要自定义。这称为「三之法则」（Rule of Three）。在 C++11 及之后，加上移动构造和移动赋值，扩展为「五之法则」（Rule of Five）。
{% endhint %}

## 递增与递减运算符

`++` 和 `--` 有前置和后置两种形式，通过参数列表中一个无用的 `int` 参数区分：

```cpp
class Counter {
private:
    int value;

public:
    Counter(int v = 0) : value(v) {}

    // 前置 ++（++c）：先自增，返回自增后的引用
    Counter& operator++() {
        ++value;
        return *this;
    }

    // 后置 ++（c++）：先保存旧值，自增，返回旧值的副本
    Counter operator++(int) {    // int 参数仅用于区分，不使用
        Counter old = *this;
        ++value;
        return old;
    }

    int getValue() const { return value; }
};

int main() {
    Counter c(5);
    std::cout << (++c).getValue() << std::endl;   // 6（前置）
    std::cout << (c++).getValue() << std::endl;   // 6（后置，返回旧值）
    std::cout << c.getValue() << std::endl;        // 7
    return 0;
}
```

{% hint style="info" %}
前置版本返回引用（效率更高），后置版本返回副本（需要保存旧值）。在不需要旧值的场景下，优先使用前置递增 `++i` 而非后置 `i++`。
{% endhint %}

## 下标运算符

`operator[]` 用于实现数组风格的元素访问，通常需要提供 const 和非 const 两个版本：

```cpp
#include <iostream>
#include <stdexcept>

class IntArray {
private:
    int* data;
    int size;

public:
    IntArray(int n) : size(n), data(new int[n]()) {}
    ~IntArray() { delete[] data; }

    // 非 const 版本：可读可写
    int& operator[](int index) {
        if (index < 0 || index >= size) {
            throw std::out_of_range("下标越界");
        }
        return data[index];
    }

    // const 版本：只读
    const int& operator[](int index) const {
        if (index < 0 || index >= size) {
            throw std::out_of_range("下标越界");
        }
        return data[index];
    }

    int getSize() const { return size; }
};

int main() {
    IntArray arr(5);
    arr[0] = 10;           // 调用非 const 版本
    arr[1] = 20;

    const IntArray& ref = arr;
    std::cout << ref[0] << std::endl;  // 调用 const 版本
    // ref[0] = 30;        // 编译错误！const 版本返回 const 引用

    return 0;
}
```

## 流插入与流提取运算符

`<<` 和 `>>` 的左操作数是 `std::ostream` / `std::istream`，不是自定义类型，因此**必须**用非成员函数重载，通常声明为友元：

```cpp
#include <iostream>

class Vector2D {
private:
    double x, y;

public:
    Vector2D(double x = 0, double y = 0) : x(x), y(y) {}

    // 流插入运算符（输出）
    friend std::ostream& operator<<(std::ostream& os, const Vector2D& v) {
        os << "(" << v.x << ", " << v.y << ")";
        return os;       // 返回流引用，支持链式调用
    }

    // 流提取运算符（输入）
    friend std::istream& operator>>(std::istream& is, Vector2D& v) {
        is >> v.x >> v.y;
        return is;
    }
};

int main() {
    Vector2D v(3.0, 4.0);
    std::cout << "向量：" << v << std::endl;   // 向量：(3, 4)

    Vector2D v2;
    std::cout << "请输入 x y：";
    std::cin >> v2;
    std::cout << "你输入了：" << v2 << std::endl;

    return 0;
}
```

要点：

- `operator<<` 接受 `const` 对象引用（输出不修改对象）
- `operator>>` 接受非 `const` 引用（输入需要修改对象）
- 两者都返回流的引用，以支持 `std::cout << a << b` 这样的链式写法

## 函数调用运算符

重载 `operator()` 可以让对象像函数一样被调用，这样的对象称为**函数对象**（Function Object）或**仿函数**（Functor）：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

class Multiplier {
private:
    int factor;

public:
    Multiplier(int f) : factor(f) {}

    int operator()(int x) const {
        return x * factor;
    }
};

int main() {
    Multiplier triple(3);
    std::cout << triple(5) << std::endl;     // 15
    std::cout << triple(10) << std::endl;    // 30

    // 函数对象可以传给 STL 算法
    std::vector<int> nums = {1, 2, 3, 4, 5};
    std::vector<int> result(nums.size());
    std::transform(nums.begin(), nums.end(), result.begin(), Multiplier(2));

    for (int n : result) {
        std::cout << n << " ";    // 2 4 6 8 10
    }
    std::cout << std::endl;

    return 0;
}
```

函数对象的优势在于它可以携带状态（如上例中的 `factor`），而普通函数无法做到。函数对象在 STL 算法中广泛使用，第十三章将详细讨论。

## 完整示例：Vector2D 类

将本节介绍的运算符整合到一个完整的类中：

```cpp
#include <iostream>
#include <cmath>

class Vector2D {
private:
    double x, y;

public:
    Vector2D(double x = 0, double y = 0) : x(x), y(y) {}

    double getX() const { return x; }
    double getY() const { return y; }

    double length() const {
        return std::sqrt(x * x + y * y);
    }

    // 复合赋值
    Vector2D& operator+=(const Vector2D& rhs) {
        x += rhs.x; y += rhs.y;
        return *this;
    }

    Vector2D& operator-=(const Vector2D& rhs) {
        x -= rhs.x; y -= rhs.y;
        return *this;
    }

    // 一元取负
    Vector2D operator-() const { return Vector2D(-x, -y); }

    // 标量乘法
    Vector2D operator*(double s) const { return Vector2D(x * s, y * s); }

    // 比较
    bool operator==(const Vector2D& rhs) const {
        return x == rhs.x && y == rhs.y;
    }
    bool operator!=(const Vector2D& rhs) const { return !(*this == rhs); }

    // 流输出
    friend std::ostream& operator<<(std::ostream& os, const Vector2D& v) {
        return os << "(" << v.x << ", " << v.y << ")";
    }

    // 友元：标量 * 向量
    friend Vector2D operator*(double s, const Vector2D& v);
};

// 非成员函数
Vector2D operator+(Vector2D lhs, const Vector2D& rhs) { return lhs += rhs; }
Vector2D operator-(Vector2D lhs, const Vector2D& rhs) { return lhs -= rhs; }
Vector2D operator*(double s, const Vector2D& v) { return v * s; }

int main() {
    Vector2D a(1, 2), b(3, 4);

    std::cout << "a + b = " << (a + b) << std::endl;    // (4, 6)
    std::cout << "-a = " << (-a) << std::endl;           // (-1, -2)
    std::cout << "a * 3 = " << (a * 3) << std::endl;    // (3, 6)
    std::cout << "2 * b = " << (2.0 * b) << std::endl;  // (6, 8)
    std::cout << "|b| = " << b.length() << std::endl;   // 5

    a += b;
    std::cout << "a += b → " << a << std::endl;          // (4, 6)

    return 0;
}
```
