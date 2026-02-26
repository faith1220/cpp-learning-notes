# 12.2 关联容器

如果在你的程序里，元素不应该只是按存入顺序排成长队，而是带有某种查找特性——也就是你希望根据特定的“键（Key）”来快速找到对应的“值（Value）”，或者你想让元素在存入时就自动排好序——你需要**关联容器（Associative Containers）**。

传统的关联容器底层一律采用**红黑树（Red-Black Tree，一种平衡二叉搜索树）**来实现，因此它们具有天生的有序性。

## 1. std::set / std::multiset（集合）

集合是一种只能存**唯一（不能重复）**元素的容器，且元素一旦存入，**不仅按升序自动排好，而且不能被直接修改**（因为改了会破坏红黑树规则）。

- **底层结构**：红黑树
- **主要特点**：元素会自动排序，不允许有重复元素，查找/插入/删除的时间复杂度都是对数级 $O(\log N)$。
- **常见用途**：去重、快速判断一个元素存在与否。

```cpp
#include <iostream>
#include <set>

int main() {
    std::set<int> mySet;

    // 插入元素
    mySet.insert(30);
    mySet.insert(10);
    mySet.insert(20);
    mySet.insert(20); // 尝试插入重复元素，会被忽略！

    // 自动按 10, 20, 30 排好序了
    for (int n : mySet) {
        std::cout << n << " ";
    }
    std::cout << "\n";

    // 查找元素
    if (mySet.find(20) != mySet.end()) {
        std::cout << "找到了 20!\n";
    }

    return 0;
}
```

> `std::multiset` 和 `set` 十分类似，唯一的区别是：`multiset` 允许存放重复的元素。

## 2. std::map / std::multimap（字典/映射）

`std::map` 就是大家熟悉的键值对（Key-Value Pair）字典。它里面装的元素实际上是一个 `std::pair<const Key, Value>`。

在 `map` 中，**所有的键（Key）都必须是唯一的，且整个容器会根据 Key 自动排序**。

- **底层结构**：红黑树
- **主要特点**：按键自动排序，键不可重复，可以用 `[]` 运算符快速按键读取/创建值。
- **常见用途**：词频统计、需要保证有序存储字典对象的场景。

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    // 键为 string，值为 int 的 map
    std::map<std::string, int> ages;

    // 插入数据：使用 [] 就是最爽直的写法
    ages["Alice"] = 24;
    ages["Bob"] = 30;
    ages["Charlie"] = 21;

    // 修改数据
    ages["Alice"] = 25; // 之前的值被覆盖

    // 查找并遍历（会按照 A, B, C 的字母序列自动排序!）
    // 这里的 pair 的类型是 std::pair<const std::string, int>
    for (const auto& pair : ages) {
        std::cout << pair.first << " is " << pair.second << " years old.\n";
    }

    // 坑点警告：如果你用 [] 去找一个不存在的键，map 会自动创建一个默认值(填0或者空)然后返回！
    // 安全的做法是使用 .count() 或者 .find() 做前置检查
    if (ages.count("David") == 0) {
        std::cout << "David 不在字典中。\n";
    }

    return 0;
}
```

> `std::multimap` 允许拥有多个相同的 Key。注意，由于 Key 不再具有唯一性，`multimap` 是**不支持** `[]` 操作符的。

## 关联容器的排序规则

我们说关联容器默认是对元素进行排序的（升序）。这是因为它们默认使用 `std::less<T>` 这个函数对象来比大小，要求你的元素（也就是 T 的类型）必须支持了 `<` 运算符重载。

如果你向 `set` 中放入自己编写的 `Student` 类对象，编译器会报错的（它不知道这俩对象谁大谁小咋排）。解决办法是给 `Student` 重载 `operator<`，或者在定义 `set` 时显式提供一个定制比较器模板参数。