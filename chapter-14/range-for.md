# 14.2 范围 for 循环（Range-based for loop）

在没有范围 for 循环之前，只要我们需要遍历一个数组、`std::vector` 或者任何容器，我们几乎都躲不开两种写法：

1. 老式基于索引的循环：`for(int i=0; i<vec.size(); i++)`（对不支持 `[]` 操作符的 `list` 和 `map` 无能为力）
2. 繁琐的迭代器循环：`for(vector<int>::iterator it = vec.begin(); it != vec.end(); ++it)`

C++11 引入了更加贴近自然语言（类似 Python, Java 等语言中增强型 for 循环）的语法结构：**基于范围的 for 循环**。

## 基本使用

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};

    // "对 numbers 里面的【每一个元素 n】做某事"
    for (int n : numbers) {
        std::cout << n << " ";
    }
    // 输出: 1 2 3 4 5
}
```

## 配合 auto 食用更佳

在实际开发中，我们甚至懒得去想 `numbers` 里装的到底是 `int` 还是什么别的东西，直接掏出 `auto` 即可。这也规避了某些微小的类型隐式转换开销。

```cpp
std::vector<int> numbers = {1, 2, 3, 4, 5};

for (auto n : numbers) {
    // ...
}
```

## 核心坑点：按值拷贝 vs 按引用传递

在使用范围 for 遇到大对象或者需要修改容器内元素时，非常容易掉入**默认值拷贝**的陷阱。

### 错误示范（试图修改原始容器无果：
```cpp
// 你的本意：想把 numbers 里的所有数字全变 0
std::vector<int> numbers = {1, 2, 3};

for (auto n : numbers) {
    n = 0; // 错误！这里的 n 被 auto 推导丢弃了引用，它只是原生里面取出的一个【副本】！
}
// numbers 仍旧是 {1, 2, 3}
```

### 正确的修改方式（使用 `auto&`）：
```cpp
for (auto& n : numbers) {
    n = 0; // 这次 n 就是原始元素的别名（引用），真被改了
}
```

### 最推荐的安全只读方式（使用 `const auto&`）：
如果你压根不打算改元素，只是从头到尾读一遍打印一遍，那就用最安全的写法。
**特别是针对存储含有大型对象（如 `std::string` 或结构体）的容器，如果是普通的 `auto n` 将对每一个对象触发缓慢的深拷贝构造函数！这是极其严重且隐蔽的性能泄露点。**

```cpp
std::vector<std::string> heavyData = {"abc.....", "def....."};

// 【最佳实践】零开销，只读安全遍历
for (const auto& str : heavyData) {
    std::cout << str << "\n";
}
```

## 它为什么能工作？

在编译器眼中，你写的 `for (auto e : container)`，在背后被它翻译成了一长串：

```cpp
auto && __range = container;
for (auto __begin = __range.begin(), __end = __range.end(); __begin != __end; ++__begin) {
    auto e = *__begin;
    // ... 你的代码
}
```

这就解释了为什么基于范围的 for 循环必须要求背后的对象（或者是你的自定义类）暴露并实现了可用的 `.begin()` 和 `.end()` 成员函数。