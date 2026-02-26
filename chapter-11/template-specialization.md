# 11.3 模板特化

模板的设计初衷是“一套代码，通吃百家类型”。但这有时候会导致问题——对绝大多数类型成立的通用逻辑，对于某些特殊的类型可能显得效率低下，甚至根本无法编译或者得出错的逻辑结果。

为了解决这个问题，C++ 提供了**模板特化（Template Specialization）**的机制，允许我们为某些特定类型提供定制版本的实现。

## 全特化（Full Specialization）

如果对于某个具体类型，你想彻底改变模板的行为，就可以使用**全特化**。特化版本必须放在通用版本之后声明。

假设我们写了一个全通用版的数字比较模板：

```cpp
#include <iostream>

// 通用版本模板
template <typename T>
bool isEqual(T a, T b) {
    return a == b;
}
```

如果你给它传入字符串 `const char*`：
```cpp
const char* s1 = "hello";
const char* s2 = new char[6]{'h','e','l','l','o','\0'};
isEqual(s1, s2); // 糟糕！这是比较指针地址，结果将是 false！
```

我们需要通过特化改变这种行为：

```cpp
#include <cstring>

// 为 const char* 提供“全特化”版本
// template <> 表示没有任何泛型参数了，类型已经被完全指定。
template <>
bool isEqual<const char*>(const char* a, const char* b) {
    return std::strcmp(a, b) == 0; // 逐字符比较真正的字符串内容
}

int main() {
    // 编译器发现是 const char*，会优先匹配特化版本
    isEqual("hello", s2); // 返回 true
}
```

## 类模板的特化

类模板的特化和函数相似。特化版本可以拥有与通用版本完全不同的成员变量和方法。这也是极为常见的手段（由于篇幅，我们只展示语法基础）。

这里有一个通用存放东西的容器，但如果是放 `bool` 类型数据，我们希望做位压缩优化（类似于 `std::vector<bool>` 的特殊机制）：

```cpp
// 1. 通用版类模板
template <typename T>
class Storage {
public:
    void print() { std::cout << "我是通用版本的存取实现\n"; }
};

// 2. 针对 bool 的全特化类模板
template <>
class Storage<bool> {
public:
    // 这里不仅方法可以变，里边的代码和成员都完全可以是另一副模样
    void print() { std::cout << "我是 bool 的极致压缩优化存取实现！\n"; }
};
```

## 偏特化（Partial Specialization）

全特化是所有泛型参数都被指定死了（剥夺了所有的灵活性）。
与之对应的是**偏特化（局部特化）**，它依然保留了一部分泛型，但对类型加了一些匹配条件限制。

{% hint style="danger" %}
**注意：C++ 仅支持对类模板进行偏特化，不支持函数模板的偏特化。**
如果要实现类似函数偏特化的效果，通常通过函数重载来替代。
{% endhint %}

偏特化一般用于指针类型（剥离指针属性）、或者有多模板参数但只确定其中一个的场景。

```cpp
// 通用版本
template <typename T>
class Printer {
public:
    void doPrint(T val) { std::cout << "Value: " << val << "\n"; }
};

// 偏特化版本：专门针对任何类型的“指针” T*
template <typename T>
class Printer<T*> {
public:
    // 特别处理如果外界传入了一个指针，我们不是打印地址，而是解引用它
    void doPrint(T* ptr) {
        if(ptr) std::cout << "Pointer points to: " << *ptr << "\n";
        else std::cout << "Null pointer\n";
    }
};

int main() {
    Printer<int> p1;
    p1.doPrint(42);         // 匹配通用版本

    int num = 100;
    Printer<int*> p2;
    p2.doPrint(&num);       // 匹配指针偏特化版本
}
```

在上面的偏特化版本中，`template <typename T>` 依然存在，说明它仍然是个没实例化的模板。但紧接其后的类名叫 `Printer<T*>`，这一匹配规则告诉编译器：“不管传入什么类型，只要它是某个类型的指针，就用这套图纸！”