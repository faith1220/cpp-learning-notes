# 15.5 折叠表达式（Fold Expressions）

我们在[11.4 变参模板](../chapter-11/variadic-template.md)中曾经带大家前瞻体验过一次折叠表达式。这是 C++17 在核心模板元编程下放的大绝招。

## C++11 写法的痛楚

为了能利用“变参参数包（Parameter Pack `...`）”拆解数量未知的入参，过去唯一的写法就是**递归展开法**：你需要硬性配置一把“终止条件重载函数接口”和一把“递归剥离参数函数接口”。哪怕只是想给所有的参数求个和，都要写一大堆结构：

```cpp
// C++11 实现 sum_all
// 1. 基线终点退出条件函数
template<typename T>
T sum_all(T val) {
    return val;
}
// 2. 剥离头一个参数递归的模板主体
template<typename T, typename... Rest>
T sum_all(T first, Rest... rest) {
    return first + sum_all(rest...);
}
```

这不仅代码难读冗长，编译时生成极深恐怖函数调用层级往往还会给开发带来负担。

## C++17 的优雅：折叠

C++17 引入**折叠表达式**，它允许你在模板包上**直接**连载运用二元运算符（如 `+`, `*`, `&&`, `<<` 等 32 种常规运算符）。而且最重要的是：它会在源编译代码层级直接内联**展平**，没有任何递归套娃代价。

```cpp
#include <iostream>
#include <string>

// C++17：折叠表达式（一元右折叠形态）
// (args + ...) 等于在脑内被直接替换为 (arg1 + (arg2 + (arg3 + arg4)))
template <typename... Args>
auto sum_all(Args... args) {
    return (args + ...);
}

// 还能配合流式输出（结合了逗号运算符或者直接 << 叠加）
// 此处采用一元左折叠
template <typename... Args>
void print_all(Args... args) {
    // 等价于: (std::cout << arg1) << arg2 << arg3...
    (std::cout << ... << args) << "\n";
}

int main() {
    std::cout << sum_all(1, 2, 3, 4, 5) << "\n";   // 15
    std::string s1 = "C++17 ", s2 = "is ", s3 = "awesome!";
    std::cout << sum_all(s1, s2, s3) << "\n";      // 因为有重载 operator+ ，完美拼装

    print_all(1, " => ", 3.14, " is amazing");
}
```

## 折叠的四种形式

你可能注意到了 `...` 放左边和放右边的细微区别。它这牵扯到折叠时向左关联合还是向右关联的概念：

1. **一元右折叠：` (pack op ...) `** 展开为: `E1 op (E2 op (E3 op E4))`
2. **一元左折叠：` (... op pack) `** 展开为: `(((E1 op E2) op E3) op E4)`
3. **二元右折叠：` (pack op ... op init) `** （带有初始值，不会在空包时报错而死卡）
4. **二元左折叠：` (init op ... op pack) `** 展开为: `(((init op E1) op E2) op E3)`

### 附赠：经典逗号包裹招式

如果我们既不像把东西加合算总账，也不想把它们当成流全喷涌出去。我们只是单纯想针对变参传进来的那几个对象，依次调用这个对象自己拥有的比如 `.close()` 方法，该怎么折叠呢？

这时候巧妙运用只保证顺序但不求结果总合的**逗号 ` , ` 折叠**：

```cpp
template <typename... Handlers>
void close_all(Handlers&... handlers) {
    // 用括号和逗号，这行代码在脑内里会被当做按序列把所有的参数都排解出来，挨个执行 `.close()` 这个成员方法！
    (handlers.close() , ...);
}
```
折叠表达式使得曾经属于底层框架师才能驾驭的元编程变参大作法，开始走入寻常日常业务组人员的工程代码。