# 13.3 Lambda 表达式

在上一节我们学习 `<algorithm>` 时看到，很多算法后面都需要我们提供一种**筛选条件**或者**自定义排序规则**。

在 C++11 以前，为了做一次普通的自定义排序，你要专门去类的外面定义一个独立的普通函数或者麻烦的“仿函数（重载了 `operator()` 的类）”才能传进去。这严重拉长了代码的物理距离，打断了阅读思路。

为了解决这个问题，C++11 引入了闭包结构——**Lambda 表达式（Lambda Expressions）**，它是现代 C++ 中最亮眼的语法糖。

## 什么是 Lambda？

你可以把 Lambda 当作是一个**匿名的、能就地编写的、能直接捕获周遭环境变数值的微型函数**。

它的一般形式被称为“Lambda 三段式”：

```cpp
[捕获列表](参数列表) -> 返回类型 { 函数体 }
```
*(注意：`-> 返回类型` 往往可以被自动推导，所以通常被省略我们不写。)*

## 基本使用与排序

让我们结合上一节的知识，看看它是如何完美服务于算法的。假设我们要给一堆字符串按包含的字母长度排序（默认 `std::sort` 是按字母字典序的 A, B, C）：

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>

int main() {
    std::vector<std::string> words = {"Apple", "Banana", "Cat", "Dinosaur"};

    // 传统默认排序是按字典序列，下面我们用 Lambda 定制：谁的字更少排在前面
    std::sort(words.begin(), words.end(), [](const std::string& a, const std::string& b) {
        return a.length() < b.length(); // a 更小说明应在 b 的左边
    });

    for (const auto& w : words) std::cout << w << " ";
    // 输出: Cat Apple Banana Dinosaur
}
```

只需要 `[](...){...}` 这么几行代码块，整个逻辑一气呵成、干净利落。

## 神奇的”捕获列表“ `[ ]`

Lambda 的牛逼之处不仅在于它是个函数，更在于那个最初出现的方括号 `[ ]`。**捕获列表决定了 Lambda 内部能否看见（并且使用）Lambda 外部在它诞生那一刻的局部变量。**

1. **`[]` 空捕获**：瞎子，我不认识外面的世界，只能用传进来的函数参数。
2. **`[x]` 值捕获**：偷偷复制了一份外面 `x` 此时此刻的值藏在身上。**内部不可修改**（若硬要修改，需要在后面加 `mutable` 关键字，但改的也仅仅是那个隐藏在这个匿名微型类自己内部的值的副本，不影响真正的外界 `x`）。
3. **`[&x]` 引用捕获**：极其强大（也稍有危险）。直接获得了外界 `x` 本体变量的操作权。内部改了，外面就真改了。
4. **`[=]` 全自动值捕获**：把 Lambda 体里面用到的所有外部可见的变量都偷偷拷一份。
5. **`[&]` 全自动引用捕获**：把 Lambda 体里面用到的所有外部可见的变量本体的操作权都拿下了。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> numbers = {10, 25, 40, 50, 75};

    int threshold = 35;  // 阈值变量，外部的！
    int count = 0;       // 统计记账的

    // 我们想找出有几个比 threshold 大的数。
    // 但是 any_of/for_each 给的方法签名只能接受 1 个代表集合体内元素的参数。
    // 没法把 threshold 当参数传。使用 [threshold] 捕获！同时用 [&count] 去修改外面那个账本！
    std::for_each(numbers.begin(), numbers.end(), [threshold, &count](int n) {
        if (n > threshold) {
            count++; // 修改引用，真改了外边的值
        }
    });

    std::cout << "超过 " << threshold << " 的数字共有 " << count << " 个。\n";
}
```

> 上述操作实际上如果用原生的 `std::count_if` 是最优雅的：
> `int count = std::count_if(vec.begin(), vec.end(), [threshold](int n){ return n > threshold; });`

## 泛型 Lambda（C++14）

在 C++14 里，可以在参数列表中使用 `auto` 关键字。此时的 Lambda 变成了一个模板函数，进一步消灭冗长拖沓的参数名声明！

```cpp
// 极简写法，不用在乎内部元素是 int 还是 string
auto myPrint = [](const auto& val) {
    std::cout << val << "\n";
};

// 甚至可以直接将 Lambda 当变量存起来到处扔！
std::vector<int> vi = {1, 2};
std::vector<std::string> vs = {"A"};

std::for_each(vi.begin(), vi.end(), myPrint);
std::for_each(vs.begin(), vs.end(), myPrint);
```

{% hint style="danger" %}
**注意！对局部变量引用的生命周期**
使用引用捕获 `[&]` 时要倍加小心：如果这个 Lambda 被别的模块存了起来推迟执行（甚至扔到了另一个线程里），如果将来这方 Lambda 开始真正运行的时候，之前提供变量的主函数域已经退出了销毁了局部变量。你此时手握的就是个**悬垂引用**的恐怖炸弹！在这种异步场景下，必须老老实实地用值捕获 `[=]` 或移动捕获。
{% endhint %}