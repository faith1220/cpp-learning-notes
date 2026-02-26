# 12.3 无序容器

在 C++11 中，标准库终于迎来了重大更新：**基于哈希表（Hash Table）实现的无序容器**。

在其他主流语言里（比如 Python 的 `dict`，Java 的 `HashMap`，C# 的 `Dictionary`），大家口中常说的“字典”往往就是哈希表。C++ 也不例外，这四个“新”成员是针对传统红黑树关联容器的强力对标产物。

## 无序容器家族

这四个容器在名字上都有个 `unordered_` 前缀：

1. `std::unordered_set`（无序集合，对应 `set`）
2. `std::unordered_map`（无序映射，对应 `map`）
3. `std::unordered_multiset`
4. `std::unordered_multimap`

## 有序（树）与无序（哈希表）的终极对比

在绝大多数“查找键名拿键值”的日常工程场景下，**优先使用 `unordered_map` 和 `unordered_set`！**

为什么呢？看一下时间复杂度：

| 容器类型 | 内部数据结构 | 元素是否有序？ | 查找 / 插入时间复杂度 | 时间性能 |
| :--- | :--- | :--- | :--- | :--- |
| 普通 `map`/`set` | 红黑树 | **有必然顺序** | $O(\log N)$ | 较慢 |
| `unordered_map/set` | 哈希表 + 桶 | **绝对无序，很乱** | 平均 $O(1)$，极差退化 $O(N)$| **极快** |

```cpp
#include <iostream>
#include <unordered_map>
#include <string>

int main() {
    std::unordered_map<std::string, int> scores;

    scores["Alice"] = 95;
    scores["Bob"] = 80;
    scores["Charlie"] = 90;

    // 你会发现遍历输出的顺序和你的插入顺序截然不一样
    // 甚至不同运行时间下的顺序都有可能因为哈希算法而变动！
    for (const auto& pair : scores) {
        std::cout << pair.first << ": " << pair.second << "\n";
    }

    return 0;
}
```

## 使用限制与哈希函数

哈希表的原理是通过一个特定的数学公式（哈希函数 `Hash(Key)`），把 Key 转换成一个数组下标进行直接存储，这也就是它 $O(1)$ 神话般速度的来源。

因此，当你试图用自己编写的类型作为 `unordered_map` 的 Key 时，你有两重任务要满足（普通 `map` 只需要满足 `<` 的重载即可）：

1. 提供一种**判断相等的方法**（系统必须确认这两个键是一致的。也就是重载 `operator==`）。
2. 提供一种**哈希计算器**函数对象（将你的自定义类型变成一个巨型整数，也就是 `std::hash<T>` 函数的实现/特化）。

对于标准库里的内置类型如 `int`、`std::string` 等，C++ 早已为你写好了这些基础能力，所有可以直接使用。

{% hint style="info" %}
**总结：在 `O(log N)` 与 `O(1)` 之间做抉择**
- 如果你**不关心**元素按什么顺序排，只是单纯地想要根据 Key 存数据和提取数据，请用 `unordered_map`。速度为王。
- 如果你有强烈的业务需求说：”我想找到所有大于 30 的人“或者“我希望拿出来的数据都是按字母排好序的”，请回头使用经典的红黑树 `map`。
{% endhint %}