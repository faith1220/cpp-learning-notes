# 10.3 shared_ptr 与 weak_ptr

虽然 `std::unique_ptr` 能解决大部分内存管理问题，但是当我们确实需要在多个地方**共享**同一个对象的生命周期和所有权时，`unique_ptr` 就不适用了。这时候我们需要 `std::shared_ptr`。

## 什么是 shared_ptr？

`std::shared_ptr` 是基于**引用计数（Reference Counting）**机制的智能指针。
- 其内部包含了一个用来记录“有多少个 shared_ptr 正在指向这块内存”的计数器（Control Block）。
- 每次复制（Copy）一个 `shared_ptr`，计数器加 1。
- 每次销毁或重新指向一个 `shared_ptr`，计数器减 1。
- **当计数器变为 0 时，指向的动态内存才会真正被 `delete`**。

```cpp
#include <iostream>
#include <memory>

class Character {
public:
    Character(std::string name) : name(name) { std::cout << name << " 创建\n"; }
    ~Character() { std::cout << name << " 销毁\n"; }
    std::string name;
};

int main() {
    std::shared_ptr<Character> p2;
    {
        // 推荐使用 std::make_shared
        std::shared_ptr<Character> p1 = std::make_shared<Character>("Arthur");
        std::cout << "计数: " << p1.use_count() << "\n"; // 输出 1

        p2 = p1; // 共享所有权（发生复制）
        std::cout << "计数: " << p1.use_count() << "\n"; // 输出 2
    } // 离开作用域，p1 销毁，计数减为 1。由于不是 0，内存不会释放！

    std::cout << "p1 被销毁，但主角还在: " << p2->name << "\n";
    std::cout << "计数: " << p2.use_count() << "\n"; // 输出 1

    return 0;
} // 离开作用域，p2 被销毁，计数减为 0。触发 Character 的析构函数
```

{% hint style="warning" %}
**重要提示：** 始终使用 `std::make_shared` 创建 `shared_ptr`，而不是用 `new` 操作符配合构造函数。`make_shared` 会将对象和控制块（记录引用计数的地方）分配在同一块连续内存中，效率更高，并且能防止内存泄漏风险。
{% endhint %}

## 循环引用问题（Circular Reference）

`shared_ptr` 有一个致命的弱点：**循环引用**会使得对象永远无法被释放。

想象一下双向链表或者相互引用的对象：

```cpp
class Person {
public:
    std::string name;
    std::shared_ptr<Person> friend_ptr; // 每个人指向对方

    Person(std::string n) : name(n) { std::cout << name << " created\n"; }
    ~Person() { std::cout << name << " destroyed\n"; }
};

int main() {
    auto p1 = std::make_shared<Person>("Alice");
    auto p2 = std::make_shared<Person>("Bob");

    p1->friend_ptr = p2; // Alice 的内部指针指向 Bob
    p2->friend_ptr = p1; // Bob 的内部指针指向 Alice
    // 发生了循环引用

    return 0;
}
```

在上例中：
1. 退出了作用域，局部变量 `p1`、`p2` 被销毁。
2. 属于 Alice 内存块的计数器因为 `p1` 销毁减一。但是 Bob 内部的 `friend_ptr` 还指向 Alice，所以 Alice 无法被释放（计数仍是 1）。
3. 同理，属于 Bob 内存块的计数器因为 `p2` 销毁减一，但 Alice 内部依然指向 Bob，所以 Bob 无法被释放（计数仍是 1）。

结果是：`Person` 的析构函数永远不会被调用，产生了内存泄漏！

## 使用 weak_ptr 解决循环引用

为了打破这种尴尬局面，C++11 引入了 `std::weak_ptr`。

`weak_ptr` 被设计为一种扮演**“旁观者”**的弱指针。它可以监控和访问 `shared_ptr` 管理的对象，但是**它不会增加引用计数！**

当对象只被 `weak_ptr` 指向，而没有被任何 `shared_ptr` 指向时，对象就会被正常析构。我们可以用 `weak_ptr` 打破上面的死循环：

```cpp
class Person {
public:
    std::string name;
    // 使用 weak_ptr 替代 shared_ptr 来打破循环
    std::weak_ptr<Person> friend_ptr;

    Person(std::string n) : name(n) { std::cout << name << " created\n"; }
    ~Person() { std::cout << name << " destroyed\n"; }
};

int main() {
    auto p1 = std::make_shared<Person>("Alice");
    auto p2 = std::make_shared<Person>("Bob");

    p1->friend_ptr = p2; // weak_ptr 不增加 p2 的计数
    p2->friend_ptr = p1; // weak_ptr 不增加 p1 的计数

    return 0;
} // Alice 和 Bob 的析构函数正常先后调用，没有内存泄露！
```

### 如何使用 weak_ptr？

因为 `weak_ptr` 引用的对象可能已经销毁了，它没法直接使用 `->` 或 `*` 访问内存。为了安全访问，你必须调用它的 `.lock()` 方法，它会尝试升级并返回一个 `shared_ptr`。如果原来的对象已经被销毁了，返回的将会是一个空的（nullptr）`shared_ptr`。

```cpp
// 假设 friend_ptr 是 std::weak_ptr
if (auto sharedFriend = p1->friend_ptr.lock()) {
    std::cout << "朋友还在，名字是: " << sharedFriend->name << "\n";
} else {
    std::cout << "朋友对象已经被销毁了。\n";
}
```

## 智能指针选择指南

总结起来，在实际项目中选择智能指针时，遵循以下规律：
1. **优先使用 `std::unique_ptr`**：绝大多数对象都有一个明确的单一拥有者。最快、最轻量级。
2. **只有在确实需要“共享权责”的时候才使用 `std::shared_ptr`**。它的内部会涉及原子操作（为了线程安全的引用计数），有一定的性能开销。
3. **结合使用 `std::weak_ptr` 来打破循环引用**。如果 A 和 B 互相持有对方，其中一方应该是 `shared_ptr`，反向的应该用 `weak_ptr`（比如父节点 `shared_ptr` 持有子节点，而子节点 `weak_ptr` 指向父节点）。