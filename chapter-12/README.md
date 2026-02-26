# 第十二章 STL 容器

标准模板库（Standard Template Library，简称 STL）是 C++ 标准库中最核心、也是日常开发中使用最频繁的部分。STL 主要由三个关键组件构成：**容器（Containers）**、**迭代器（Iterators）** 和 **算法（Algorithms）**。

本章我们将聚焦于 **容器**。

容器，顾名思义，就是用来“装数据”的数据结构模板类。STL 为我们预先实现了各种极其注重性能优化的经典数据结构（如动态数组、链表、红黑树、哈希表等），让我们免于重复造轮子。

根据数据的组织方式，STL 容器主要分为四大类：

1. **[序列容器（Sequence Containers）](sequence-containers.md)**：数据按线性顺序排列（如 `std::vector`, `std::list`）。
2. **[关联容器（Associative Containers）](associative-containers.md)**：数据呈非线性排列，基于树结构，支持高效的关键字搜索（如 `std::map`, `std::set`）。
3. **[无序容器（Unordered Containers）](unordered-containers.md)**：C++11 引入，基于哈希表实现的关联容器（如 `std::unordered_map`）。
4. **[容器适配器（Container Adapters）](container-adapters.md)**：在现有容器之上进行接口限制或封装而来的特殊数据结构（如 `std::stack`, `std::queue`）。

{% hint style="success" %}
**不要造轮子！**
在 C++ 开发中，除非你有极其明确的特殊性能/内存需求，否则**永远优先使用 STL 容器**代替原生的 C 风格数组或手写数据结构。它们是经过全世界 C++ 专家千锤百炼的基础骨架，具有最高的安全性和可靠性。
{% endhint %}
