# 13.2 常用算法

C++ 标准库在 `<algorithm>` 和 `<numeric>` 中内置了上百个经过极度性能优化的通用算法。

它们接收一对迭代器 `[first, last)` 表示要处理的范围，并且不关心背后的真实容器是什么。

下面我们分类总结在日常开发中使用频率极高的一些经典算法“明星”，熟练掌握它们能大幅度削减手写 `for` 循环的重复代码。

## 查找类算法
目的：在这个区间里，东西在不在？在哪？有多少个？

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> v = {10, 20, 30, 40, 20, 50};

    // 1. std::find (返回第一个匹配的迭代器，找不到返回 end())
    auto it1 = std::find(v.begin(), v.end(), 30);
    if (it1 != v.end()) std::cout << "找到了 30! \n";

    // 2. std::count (全揽统计有多少个一模一样的值)
    int count = std::count(v.begin(), v.end(), 20); // count = 2

    // 3. std::any_of (C++11：这堆东西里，【有没有任意一个】满足条件？)
    // 配合在后面一节立即会讲的 Lambda 表达式，超级强大！
    bool hasEven = std::any_of(v.begin(), v.end(), [](int n){ return n % 2 == 0; });

    // 4. std::all_of / std::none_of
    // 问：是不是【所有】都大于0？ 是不是【全都不】小于0？
}
```

## 变更与排序类算法
目的：打乱次序、把它们排好、或者彻底翻过来。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <random> // 不要再用 C 的 rand() 啦！

int main() {
    std::vector<int> v = {5, 2, 9, 1, 5};

    // 1. std::sort（快速排序主力）
    // （要求 RandomAccessIterator，所以适合 vector/deque/array）
    std::sort(v.begin(), v.end()); // 默认升序： 1 2 5 5 9

    // 使用系统预置的 greater 函数对象反向排序（降序）
    std::sort(v.begin(), v.end(), std::greater<int>()); // 9 5 5 2 1

    // 2. std::reverse（头尾彻底翻转）
    std::reverse(v.begin(), v.end()); // 1 2 5 5 9

    // 3. std::unique（“去重”，但只去除【相邻】的重复项！）
    // 如果要全盘去重，必须先 sort 再 unique
    // 注意：unique 并不会真删除容量大厦，它只是把重复的元素丢到队列屁股后面去！
    // 并且会返回一个新的尾迭代器，告诉你真实的有效数据在哪里截至。
    auto new_end = std::unique(v.begin(), v.end());

    // 结合原生的 erase，这就构成了 C++ 著名的 idiom（惯用法）：
    // Erase-Remove (Erase-Unique) 口诀
    v.erase(new_end, v.end()); // 真删除了
}
```

## 拷贝与变换类算法
目的：将数据从这搬到那顺便作妖，或者彻底涂满一片。

```cpp
#include <algorithm>
#include <vector>
#include <numeric>

int main() {
    std::vector<int> src = {1, 2, 3};
    // 注意：目标容器一定要自己预留足够的空间好装！
    std::vector<int> dest(3);

    // 1. std::copy
    std::copy(src.begin(), src.end(), dest.begin());

    // 2. std::transform（极重要：映射操作，Map）
    // 把源序列的每个元素，经过某个函数处理，丢给目标序列
    std::transform(src.begin(), src.end(), dest.begin(),
                   [](int x) { return x * 10; }); // dest 变成了 10, 20, 30

    // 3. std::fill (涂满)
    std::fill(dest.begin(), dest.end(), 0); // 全刷成 0
}
```

## 数值统筹类（在 `<numeric>` 中）
```cpp
#include <numeric>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3, 4};

    // 1. std::accumulate (求和汇总。可以理解为 Reduce 操作)
    // 参数3：是从哪个数字为起始基础开始累加（这是 0）
    int sum = std::accumulate(v.begin(), v.end(), 0); // 10

    // 如果起始数字是字符串类型，它就会疯狂拼接字符串（性能不太高，但可用）
}
```

在使用这些算法时，很多时候它们最后都会留一个函数指针的接口给你（比如用来提供特殊的排序判定依据：比较人的身高怎么排？），这就是下一节 Lambda 表达式要登场的舞台！