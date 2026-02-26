# 15.1 结构化绑定（Structured Bindings）

在编程中，我们经常遇到一个函数需要返回多个值的情况。
在 C++11 之前我们要么传引用的出参去改值，要么被迫定义一个临时的 `struct`。C++11 引入了 `std::tuple`（元组）和 `std::tie`，勉强缓解了这个问题但是依然冗长。

C++17 终于引入了**结构化绑定**，允许程序员一行代码、原模原样地拆解并解包多个返回值！

## 返回和解包 std::tuple / std::pair

过去，如果你调用了 `std::map::insert()`，它往往会返回一个恶心的 `std::pair<iterator, bool>` 告诉你插入在什么位置以及是否插入成功。现在的你应该这么接：

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<int, std::string> dict;

    // C++17 结构化绑定：直接使用 auto [变量名1, 变量名2] 解包
    auto [it, success] = dict.insert({1, "Apple"});

    if (success) {
        std::cout << "插入成功，键是: " << it->first << "\n";
    }

    return 0;
}
```

如果是你自己写的函数，可以搭配 `std::tuple` 用得极为舒展：

```cpp
#include <tuple>
#include <string>

// 返回一个包含三个混合类型值的元组
std::tuple<int, std::string, double> getSystemInfo() {
    return {200, "OK", 99.9};
}

int main() {
    // 一次性接住，连类型都不用管！
    auto [code, message, uptime] = getSystemInfo();

    // 如果只想读，可以加 const 修饰
    const auto [c, m, u] = getSystemInfo();
}
```

## 直接解包 struct / class

结构化绑定不仅能拆解标准库里的元组对象，它可以直接暴力“解构”你自定义的所有的含有 `public` 公开成员变量的简单结构体（或者数组）！

```cpp
#include <iostream>

struct Point {
    double x;
    double y;
    double z;
};

int main() {
    Point target{10.5, 20.1, 0.0};

    // 直接给结构体的各个 public 成员变量起个本地别名并解包
    auto [x, y, z] = target;
    std::cout << "x 坐标是: " << x << "\n";

    // 甚至可以在范围 for 循环里解包 map/字典 里的键值对！
    // 告别恶心的 pair.first 和 pair.second！
    std::map<int, std::string> users = {{1, "Alice"}, {2, "Bob"}};
    for (const auto& [id, name] : users) {
        std::cout << "ID: " << id << " Name: " << name << "\n";
    }

    return 0;
}
```

{% hint style="danger" %}
**注意引用修饰符**
结构化绑定的外层 `auto` 依然遵循第 14 章讲过的退化拷贝原则。
上面的 `auto [x, y, z] = target;` 实际上是将 `target` 整体拷贝了一份藏在暗处，然后 x, y, z 是这个暗处副本的变量引用。
如果你想真正在解包时去**修改**原来的 `target` 对象里的值，必须加上引用：`auto& [x, y, z] = target;`
{% endhint %}