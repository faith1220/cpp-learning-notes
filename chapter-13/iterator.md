# 13.1 迭代器

我们可以把迭代器（Iterator）简单理解为一种**带有面向对象特性的原生指针**。

由于 `std::list` 没有经过连续的内存分配，原生指针通过 `p++` 怎么可能找到树或链表中的下一个节点呢？迭代器类通过在内部重载了 `operator++` 和 `operator*`（解引用），完美掩盖了这背后的复杂操作。

## 迭代器的基本概念

任何 STL 容器都会自己内置几种属于它自己特性的迭代器类型。通过调用容器的 `.begin()` 和 `.end()`，我们就能获得迭代器去遍历它。

```cpp
#include <iostream>
#include <vector>
#include <list>

int main() {
    std::vector<int> vec = {1, 2, 3};
    std::list<int> lst = {4, 5, 6};

    // 使用迭代器遍历 vector
    // 迭代器类型通常很长，早期 C++ 写法： std::vector<int>::iterator it = vec.begin()
    for (auto it = vec.begin(); it != vec.end(); ++it) {
        std::cout << *it << " ";  // 解引用，输出 1 2 3
    }
    std::cout << "\n";

    // 使用迭代器遍历 list (用法一模一样！)
    for (auto it = lst.begin(); it != lst.end(); ++it) {
        std::cout << *it << " ";  // 输出 4 5 6
    }

    return 0;
}
```

{% hint style="warning" %}
**重要概念：左闭右开区间 `[begin, end)`**
`.begin()` 返回的迭代器指向容器的**第一个元素**。
`.end()` 返回的迭代器指向容器**最后一个元素的后面一个位置（尾后，Past-the-end）**。它不指向任何实际元素，不要对其进行解引用（`*it`）操作！它仅仅用作遍历结束的标志。
{% endhint %}

## 迭代器的种类

既然是为了兼容千奇百怪的容器，迭代器的能力也是分三六九等的。由于算法经常对“能不能随机跳”、“能不能往回走”有需求，现代 C++ 将迭代器主要分为了以下几种核心能力级别：

1. **输入/输出迭代器（Input/Output Iterator）**：最底层。只能从头到尾读/写一遍，不能往回走，不支持重复访问（如从流里度文字 `std::istream_iterator`）。
2. **前向迭代器（Forward Iterator）**：只能一步一步往后走（`++it`），可以读/写。典型代表：**`std::forward_list` 和 `std::unordered_map` 等单向/哈希容器**。
3. **双向迭代器（Bidirectional Iterator）**：支持 `++it` 往后走，也支持 `--it` 往前走。典型代表：**`std::list`, `std::map`, `std::set` 等双向结构或红黑树**。
4. **随机访问迭代器（Random Access Iterator）**：除了前向后退，它还支持像指针一样的“算术跳跃”，你可以写 `it += 5` 或 `it1 - it2`。典型代表：**`std::vector`, `std::deque`, `std::array` 还有原生指针（原生指针本身也是随机访问迭代器！）**

算法的参数往往会对送进来的迭代器有能力断言要求。比如 `std::sort()` 算法的参数通常会限定要求必须是 Random Access Iterator。因此，你可以 `sort(vec.begin(), vec.end())`，但不能去 `sort(list.begin(), list.end())` （因为 `list` 迭代器不支持跳跃寻找，只能用 `list` 自己的 `lst.sort()` 方法）。

## `const` 与反向迭代器

在很多场景下，我们仅需要“看看”容器，不需要改它内部的具体值。为了保证不被误修改导致 bug，标准库给了我们多种选择：

- **`cbegin()` / `cend()`**：返回的是 `const_iterator`，解引用之后拿到的是 `const T&`，不允许更改数据。
- **`rbegin()` / `rend()`**：反向迭代器（Reverse Iterator），`rbegin` 指向真实的最后一个，而 `++it` 动作实际上是在往开头移动。
- **`crbegin()` / `crend()`**：防修改的反向迭代器。

```cpp
std::vector<int> nums = {10, 20, 30};

// 从尾到头打印，常用于逆序检索
for (auto it = nums.rbegin(); it != nums.rend(); ++it) {
    std::cout << *it << " "; // 输出 30 20 10
}
```

## 迭代器失效（Iterator Invalidation）

这是 C++ 初学者极容易踩坑，甚至是高级开发人员稍不注意就会犯下的严重错误！

**当底层容器发生结构性突变（如扩容、删除）时，原先获取的迭代器就立刻成为了一张废纸（悬垂指针），对它的任何操作都会带来未定义行为（程序崩溃）！**

### 经典案例：边遍历边删除
如果你想把容器里的特定偶数删掉：

```cpp
#include <vector>

int main() {
    std::vector<int> vec = {1, 2, 2, 3, 4};

    // 严禁以下写法！因为 erase 执行后，内部元素大量搬家，it 会彻底失效！
    // for (auto it = vec.begin(); it != vec.end(); ++it) {
    //     if (*it % 2 == 0) vec.erase(it);
    // }

    // 【正确安全写法】
    for (auto it = vec.begin(); it != vec.end(); /* 注意这里不自增 */) {
        if (*it % 2 == 0) {
            // erase 的标准行为是：它会删掉数据，并返回指向刚删数据”下一个有效位置“的心迭代器！
            it = vec.erase(it);
        } else {
            ++it; // 没删东西，才自己往后走
        }
    }
}
```