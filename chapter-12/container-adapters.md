# 12.4 容器适配器

容器适配器（Container Adapters）是一类非常特殊的组件。正如“插座适配器/转换头”的作用一样，它们**自己不造新轮子**，而是将已有的基本序列容器（如 `deque`，`list`，`vector`）拿过来，将其外面的操作接口给“套死屏蔽掉”，从而只暴露出特定的极其有限的访问接口。

这样就形成了行为严格受限的新数据结构。

C++ STL 提供了三种标准的容器适配器：

## 1. std::stack（栈）

满足经典的 LIFO（Last In, First Out，后进先出）结构。只能从容器的**一端（称为栈顶）**压入和弹出数据。

- 底层默认依托的容器：`std::deque`
- 核心操作：`push()`压栈、`pop()`弹栈、`top()`返回栈顶元素、`empty()`判空。

```cpp
#include <iostream>
#include <stack>

int main() {
    std::stack<int> s;

    s.push(10); // 入栈底
    s.push(20);
    s.push(30); // 入栈顶

    // 我们没法像 vector 那样用 for 循环遍历打印栈，因为适配器屏蔽了所有迭代器和索引机制！
    // 只能遵循规则一个个抛出来
    while (!s.empty()) {
        std::cout << s.top() << " "; // 查看最上面的那个
        s.pop(); // 把最上面的踢走
    }
    // 输出: 30 20 10

    return 0;
}
```

## 2. std::queue（队列）

满足经典的 FIFO（First In, First Out，先进先出）结构。数据从队尾挤入，从队头排出（像去银行排队取号）。

- 底层默认依托的容器：`std::deque`
- 核心操作：`push()`队尾进、`pop()`队头出、`front()`看队头、`back()`看队尾。

```cpp
#include <iostream>
#include <queue>

int main() {
    std::queue<std::string> q;

    q.push("Alice"); // 排第一个
    q.push("Bob");
    q.push("Charlie"); // 排最后一个

    while (!q.empty()) {
        std::cout << "现在轮到: " << q.front() << "\n";
        q.pop(); // 服务完，请他走
    }
    // 输出: Alice, Bob, Charlie

    return 0;
} //
```

## 3. std::priority_queue（优先队列）

这是一个非常强大的利器。它虽然叫队列，但内部并不是按先来后到的顺序排列，而是类似“重症病房抢救通道”——**不论元素何时进来，始终动态保证值最大（或者设定最高优先级）的元素排在最前面！**

- 内部结构实质：**二叉堆（Binary Heap）数据结构**
- 底层默认依托的容器：`std::vector` （配以强大的 `make_heap`/`push_heap` 算法组建）
- 默认情况下是大顶堆（即数字越大的永远浮在上面第一个出来；如要小顶堆需提供自己的比较器）。
- 常用它来极速找出一个乱序流中的 TOP 1。

```cpp
#include <iostream>
#include <queue>

int main() {
    // 默认大顶堆版本
    std::priority_queue<int> pq;

    pq.push(10);
    pq.push(100);
    pq.push(5);
    pq.push(50);

    // 挨个弹出来，你会发现居然排好序了，大树在最上面
    while (!pq.empty()) {
        std::cout << pq.top() << " ";
        pq.pop();
    }
    // 输出 100 50 10 5

    return 0;
}
```

{% hint style="warning" %}
**易错提醒**
所有适配器中，`pop()` 都**只是负责无情地踢走顶端的数据，它不会把刚刚踢走的值返回给你**！（它是一个返回值类型为 `void` 的函数）。如果你又想知道到底是谁在这个顶端，又想踢它走，你必须分毫不差地连着打出这两套组合拳：先调查询接口取暂存（比如：`int val = s.top();`），然后再踢人（`s.pop();`）。
{% endhint %}