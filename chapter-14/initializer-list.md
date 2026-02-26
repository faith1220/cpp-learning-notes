# 14.3 统一初始化列表

在 C++11 之前，”如何初始化一个变量“对初学者可以说是极其混乱和精神分裂的：

- 你可以用括号：`int x(0);`, `std::string s("hello");`
- 你可以用等号：`int y = 0;`
- 初始化数组你得用大口红：`int arr[3] = {1, 2, 3};`
- 初始化动态装逼数据结构的容器更恶心，你要 `push_back` 几十行！

为了终结这场混乱，C++11 引入了**列表初始化（List Initialization）**，或者叫**统一初始化（Uniform Initialization）**。

## 使用大括号 `{ }` 一统天下

在 C++11 开始，你可以对**任何类型**的对象使用一对大括号 `{ }` 进行初始化。

```cpp
// 基本数据类型
int a{5};
double b{3.14};

// 数组
int arr[3]{1, 2, 3};
int* dynamicArr = new int[3]{1, 2, 3}; // 动态数组也可以直接指明元素！

// 结构体和类对象
struct Point { int x, y; };
Point p{10, 20};

// STL 容器从此告别循环 push_back！
#include <vector>
#include <map>

std::vector<int> v{1, 2, 3, 4, 5};
std::map<std::string, int> dict{ {"Alice", 24}, {"Bob", 30} };
```

## {} 带来的两大附加红利

### 1. 阻止窄化转换（Narrowing Conversion）

C++ 从 C 继承了一个历史遗留的静默妥协：允许大范围数值强行塞进小范围变量中发生阶段。

```cpp
int x = 3.14; // 编译器顶多给个警告，静默截断变 3
```

如果你使用大括号 `{}` 初始化，这就成了**硬编译错误**，在编译期直接扼杀丢失精度的代码：

```cpp
int x{3.14};  // [错误] 不能将 double 窄化转换到 int！
char c{1234}; // [错误] 1234 越出了 char 的上限！
```

### 2. 避免 Most Vexing Parse（最令人头疼的解析问题）

C++ 有一条非常奇葩的语法规则解析传统：**如果一条语句看起来像函数声明，编译器绝对优先把它当成函数声明！**

```cpp
class Widget {
public:
    Widget() {}
};

// 你本意是想调用默认无参构造函数产生一个叫 w 的对象：
Widget w();

// 【抱歉，编译器报错】
// 编译器把上面这一行解析成了：声明了一个名叫 w 的函数，它不接受入参，返回 Widget。
```

解决这种歧义的最佳现代做法，就是放弃令人混淆的小括号，采用空大括号（列表初始化）：

```cpp
Widget w{}; // 完全合法，毫无歧义地调用无参默认构造函数，实例化对象 w
```

## 背后的功臣：`std::initializer_list`

为什么我们可以向 `std::vector` 和 `std::map` 一口塞入数量不等的大括号里的那一串值？
这是因为 STL 的所有相关类的构造函数都被重载扩充了，提供了一个接受 `std::initializer_list<T>` 类型的参数重载！

你甚至可以在自己的类里编写接收初始化列表的方法：

```cpp
#include <iostream>
#include <initializer_list>

class MyBag {
public:
    // 当遇到 MyBag b{1,2,3,4} 时，大括号里的数组合会自动转换成 initializer_list 送给这个构造函数。
    MyBag(std::initializer_list<int> list) {
        std::cout << "收到了 " << list.size() << " 个物品\n";
        for (auto item : list) {
            // ...
        }
    }
};

int main() {
    MyBag b{1, 5, 9, 23, 40};
}
```