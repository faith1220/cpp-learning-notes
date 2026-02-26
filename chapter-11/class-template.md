# 11.2 类模板

除了函数模板，我们同样可以使得整个类的状态和行为都参数化，这就构成了**类模板（Class Template）**。这在编写通用容器（Container，如动态数组、链表、栈、队列）的时候大显身手。

## 定义类模板

我们将定义一个简单的 `Box` 类。这个 `Box` 可以用来装任何东西。

```cpp
#include <iostream>
#include <string>

// 声明一个类模板
template <typename T>
class Box {
private:
    T item;

public:
    // 构造函数
    Box(T i) : item(i) {}

    // 成员函数
    void setItem(T i) { item = i; }
    T getItem() const { return item; }
};

int main() {
    // 实例化类模板：必须使用尖括号 <> 明确给出类型！(C++17 之前)
    Box<int> intBox(100);
    Box<std::string> strBox("Hello Box");

    std::cout << intBox.getItem() << "\n";
    std::cout << strBox.getItem() << "\n";

    return 0;
}
```

### C++17 的 CTAD (类模板参数推导)

在 C++17 之前，实例化类模板必须显式指定尖括号里的类型。但从 C++17 开始引入了 **CTAD（Class Template Argument Deduction）** 特性，如果编译器能从构造函数的入参中推断出类型，就可以省略尖括号：

```cpp
// C++17 及以后可以直接这样写：
Box intBox(100);             // 自动推断出 Box<int>
Box strBox("Hello Box");     // 这里实际上推测为 Box<const char*>
```

## 类模板外联定义成员函数

如果类模板的代码很多，我们通常希望把成员函数的声明和实现分离开。在类模板外部定义成员函数时，必须带上 `template` 语法和 `<T>` 后缀。

```cpp
template <typename T>
class Pair {
private:
    T first, second;
public:
    Pair(T a, T b);
    T getMax() const;
};

// 在类外部实现构造函数
template <typename T>
Pair<T>::Pair(T a, T b) : first(a), second(b) {}

// 在类外部实现普通成员函数
template <typename T>
T Pair<T>::getMax() const {
    return (first > second) ? first : second;
}
```

{% hint style="danger" %}
**重要坑点：模板代码的编译模型**

对于普通的 C++ 类和函数，我们习惯把“声明”放在 `.h` 头文件，把“实现”放在 `.cpp` 源文件中。
**但是，模板类和模板函数不能这样做！**

由于前面提到的，模板仅仅是一张“图纸”，编译器在看到“图纸”时不会生成任何二进制代码。只有当在其它 `.cpp` 文件里真正用到 `Pair<int>` 发生实例化时，编译器才需要完整的源代码来生成对应的二进制。如果把实现放在另一个独立的 `.cpp` 中，编译器在实例化时就找不到实现体，最终导致**链接错误 (LNK2019 / Undefined reference)**。

**解决方案**：永远把模板的声明和实现都写在同一个头文件（`.h` 或 `.hpp`）中！
{% endhint %}

## 非类型模板参数

模板参数列表不仅仅可以放置“类型（typename T）”，还可以放置具体的“值”（比如整数）。这就是非类型模板参数。C++ 中最著名的例子就是 `std::array`。

```cpp
// N 是一个非类型的模板参数，它代表着数组的大小常量
template <typename T, int N>
class StaticArray {
private:
    T data[N]; // 直接分配固定大小在栈上

public:
    int getSize() const { return N; }

    // 返回引用以便修改
    T& operator[](int index) { return data[index]; }
};

int main() {
    StaticArray<int, 5> arr;
    arr[0] = 42;
    std::cout << "Array size: " << arr.getSize() << "\n";
}
```

注意：非类型参数在 C++20 之前必须是被视为编译期常量的类型（如 `int`, `enum`, 指针等），不能是浮点数或类对象。