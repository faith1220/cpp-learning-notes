# 10.2 unique_ptr

`std::unique_ptr` 是 C++11 引入的一种智能指针，用于帮助我们解决动态内存分配时需要手动 `delete` 的痛点。如果你需要动态分配内存，**默认情况下应该优先使用 `std::unique_ptr`**。

要使用智能指针，必须包含头文件 `<memory>`。

## 什么是 unique_ptr？

`std::unique_ptr` 是一种**独占所有权（Exclusive Ownership）**的智能指针。这意味着在任何给定的时刻，这段动态分配的内存只能由一个 `unique_ptr` 拥有。

当一个 `unique_ptr` 离开其作用域被销毁时，它会自动释放其包含的原始指针。这本质上就是使用了上一节学到的 RAII 原则。

## 基本用法

### 1. 创建 unique_ptr

在 C++14 之后，推荐使用 `std::make_unique` 函数来创建 `unique_ptr`，这比直接 `new` 更安全，性能也更好。

```cpp
#include <iostream>
#include <memory>

class Player {
public:
    Player() { std::cout << "Player 创建\n"; }
    ~Player() { std::cout << "Player 销毁\n"; }
    void attack() { std::cout << "攻击！\n"; }
};

int main() {
    {
        // 推荐：使用 std::make_unique（C++14 及以上）
        std::unique_ptr<Player> p1 = std::make_unique<Player>();

        // 也可以不写类型，使用 auto
        auto p2 = std::make_unique<Player>();

        p1->attack(); // 用法和普通指针完全一样
    } // 离开作用域，p1 和 p2 被销毁，自动调用 Player 的析构函数

    return 0;
}
```

### 2. 独占所有权与不可复制

因为 `unique_ptr` 是“独占”的，所以它**不允许被复制（Copy）或赋值（Assign）**。如果它能被复制，就有两个智能指针指向同一块内存，销毁时就会引发双重释放（Double Free）错误。

```cpp
std::unique_ptr<int> p1 = std::make_unique<int>(10);
// std::unique_ptr<int> p2 = p1; // 编译错误！无法复制
```

### 3. 使用 std::move 转移所有权

虽然不能复制，但我们可以**转移（Move）**所有权。使用 `std::move` 会将内存的控制权从一个 `unique_ptr` 交给另一个 `unique_ptr`。

```cpp
std::unique_ptr<int> p1 = std::make_unique<int>(100);

// 将 p1 的所有权转移给 p2
std::unique_ptr<int> p2 = std::move(p1);

// 此时 p1 变成了空指针（nullptr）
if (p1 == nullptr) {
    std::cout << "p1 已经是空指针了\n";
}

// 内存安全地由 p2 管理
std::cout << *p2 << "\n"; // 输出 100
```

## 管理数组

`unique_ptr` 也可以用来管理动态数组：

```cpp
// 创建一个包含 5 个整数的动态数组
auto arr = std::make_unique<int[]>(5);

// 可以使用 [] 访问元素
for (int i = 0; i < 5; ++i) {
    arr[i] = i * 10;
}
```

## 作为函数的参数和返回值

### 1. 传递给函数

如果你需要将独占资源传递给一个函数并在那里管理它的生命周期，你需要使用 `std::move`：

```cpp
void takeOwnership(std::unique_ptr<Player> p) {
    p->attack();
    // 离开函数时对象会被销毁
}

int main() {
    auto myPlayer = std::make_unique<Player>();
    // takeOwnership(myPlayer); // 编译错误：不能复制
    takeOwnership(std::move(myPlayer)); // 正确：转移所有权
}
```

如果函数只是想使用对象而不取得所有权，直接传递**裸指针（Raw Pointer）**或引用是最好的做法：

```cpp
// 强烈推荐：使用引用（或者用裸指针 if 允许为空）
void usePlayer(Player& p) {
    p.attack();
}

int main() {
    auto myPlayer = std::make_unique<Player>();
    usePlayer(*myPlayer); // 解引用后得到对象，不会影响所有权
}
```

### 2. 作为函数的返回值

`unique_ptr` 非常适合从工厂函数返回创建好的对象。此时编译器会自动使用移动语义，你不需要显式使用 `std::move`。

```cpp
std::unique_ptr<Player> createPlayer() {
    return std::make_unique<Player>();
}

int main() {
    auto p = createPlayer(); // 接收所有权
}
```

## 常见问题

**Q: `unique_ptr` 有性能开销吗？**
A: 几乎没有（Zero-cost abstraction）。`unique_ptr` 在内存中的大小与普通指针完全一样。其析构时的清理代码也就是普通 `delete` 的汇编指令，只不过编译器在离开作用域的地方静默自动插入了这一操作而已。

**Q: 如何让 `unique_ptr` 提前释放内存？**
A: 你可以将其置为 `nullptr` 或调用 `.reset()`：
```cpp
auto p = std::make_unique<int>(5);
p.reset(); // 释放内存
// 或者 p = nullptr;
```