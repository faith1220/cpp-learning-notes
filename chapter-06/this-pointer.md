# 6.4 this 指针

## 问题引入

当一个成员函数被调用时，它如何知道自己在操作哪个对象的数据？

```cpp
class Counter {
private:
    int count;

public:
    Counter(int count) : count(count) {}

    void increment() {
        count++;    // 哪个对象的 count？
    }
};

int main() {
    Counter a(0);
    Counter b(10);
    a.increment();   // 应该修改 a 的 count
    b.increment();   // 应该修改 b 的 count
    return 0;
}
```

答案是：每个成员函数都隐含一个指向当前对象的指针，这就是 **this 指针**。

## this 指针的本质

`this` 是一个隐式参数，类型为指向当前对象的**常量指针**。对于 `Counter` 类的非 const 成员函数，`this` 的类型是 `Counter* const`（指针本身不可修改，指向的对象可修改）。

编译器在底层会将成员函数的调用转换为类似这样的形式：

```cpp
// 你写的代码
a.increment();

// 编译器实际执行的逻辑（伪代码）
Counter::increment(&a);    // 将对象的地址作为 this 传入
```

成员函数中访问成员变量时，实际上是通过 `this->` 访问的：

```cpp
void increment() {
    count++;            // 等价于 this->count++
}
```

大多数情况下 `this->` 可以省略，编译器会自动补全。但有些场景必须显式使用 `this`。

## 场景一：参数名与成员变量同名

当构造函数或成员函数的参数名与成员变量重名时，需要用 `this->` 区分：

```cpp
class Point {
private:
    double x;
    double y;

public:
    Point(double x, double y) {
        this->x = x;   // this->x 是成员变量，x 是参数
        this->y = y;
    }

    void setPosition(double x, double y) {
        this->x = x;
        this->y = y;
    }
};
```

{% hint style="info" %}
使用成员初始化列表可以避免这个问题，因为初始化列表中 `x(x)` 的语法不会产生歧义——括号外的是成员变量，括号内的是参数。这也是推荐使用初始化列表的原因之一。
{% endhint %}

## 场景二：返回对象自身（链式调用）

通过返回 `*this`（即当前对象的引用），可以实现**链式调用**（Method Chaining）：

```cpp
#include <iostream>
#include <string>

class QueryBuilder {
private:
    std::string table;
    std::string condition;
    int limitCount;

public:
    QueryBuilder() : limitCount(0) {}

    QueryBuilder& from(const std::string& t) {
        table = t;
        return *this;    // 返回当前对象的引用
    }

    QueryBuilder& where(const std::string& cond) {
        condition = cond;
        return *this;
    }

    QueryBuilder& limit(int n) {
        limitCount = n;
        return *this;
    }

    void execute() const {
        std::cout << "SELECT * FROM " << table;
        if (!condition.empty()) {
            std::cout << " WHERE " << condition;
        }
        if (limitCount > 0) {
            std::cout << " LIMIT " << limitCount;
        }
        std::cout << std::endl;
    }
};

int main() {
    QueryBuilder query;
    query.from("users")
         .where("age > 18")
         .limit(10)
         .execute();
    // 输出：SELECT * FROM users WHERE age > 18 LIMIT 10
    return 0;
}
```

链式调用的关键在于每个方法返回 `*this`，即当前对象的引用。调用 `query.from("users")` 返回 `query` 自身的引用，因此可以继续在其上调用 `.where()`。

## 场景三：在成员函数中传递自身

有时需要将当前对象传给其他函数：

```cpp
class Widget;

void registerWidget(Widget* w);

class Widget {
private:
    std::string name;

public:
    Widget(const std::string& name) : name(name) {}

    void init() {
        // 将自身注册到某个管理器
        registerWidget(this);
    }

    std::string getName() const { return name; }
};
```

## const 成员函数中的 this

在 const 成员函数中，`this` 的类型变为 `const ClassName* const`，即指向的对象也不可修改。这就是为什么 const 成员函数无法修改成员变量：

```cpp
class Demo {
private:
    int value;

public:
    Demo(int v) : value(v) {}

    void modify() {
        this->value = 10;   // 正确，this 类型是 Demo* const
    }

    void read() const {
        // this->value = 10; // 错误！this 类型是 const Demo* const
        std::cout << this->value << std::endl;  // 只读访问，正确
    }
};
```

## this 指针的特性总结

| 特性 | 说明 |
|------|------|
| 类型 | 非 const 函数中为 `T* const`，const 函数中为 `const T* const` |
| 生命周期 | 仅在成员函数执行期间有效 |
| 不可赋值 | `this` 是右值，不能执行 `this = ...` |
| 静态函数 | 静态成员函数没有 `this` 指针（下一节讨论） |
| 显式使用 | 大多数情况可省略，名称冲突和返回自身时必须使用 |

## 常见问题

**Q：this 指针存储在哪里？**

`this` 通常通过寄存器传递（在 x86-64 架构上，常见的做法是通过 `rcx` 或 `rdi` 寄存器），而不是存储在对象内部。它不占用对象的内存空间。

**Q：this 可以为空吗？**

在合法的 C++ 代码中，`this` 不应该为空。对空指针调用成员函数是未定义行为（Undefined Behavior）。
