# 15.2 optional / variant / any

C++17 从大名鼎鼎的 Boost 库中引入了三个极度现代化的“类型包装器（Type Wrappers）”，它们彻底改变了我们表达代码意图和处理异常状态的方式。

## std::optional：表达“这里可能没有值”

以前当你写一个查找函数时，如果没找到，你会怎么返回？
- 返回 `-1`？（可如果 `-1` 也是合法的值呢？）
- 返回一个表示无效的空字符串？
- 返回一个指针结构并赋予 `nullptr`？（需要动态内存分配或者传局部变量地址的破绽）

`std::optional<T>` 完美解决了：这个对象“要么装了一个真实有效的 `T`，要么什么都没有装（`std::nullopt`）”。

```cpp
#include <iostream>
#include <optional>
#include <string>

// 查找用户的数据库请求，可能找不到
std::optional<std::string> findUser(int id) {
    if (id == 1) return "Alice"; // 直接返回实际类型，optional 自动打包
    if (id == 2) return "Bob";

    // 找不到！优雅地返回一个特定的空标志
    return std::nullopt;
}

int main() {
    auto result = findUser(3);

    // optional 居然可以直接丢进 if 判断它里面有没有值！
    if (result) {
        // 使用 .value() 取出真东西（如果为空强行取会引发异常抛出）
        // 这里也可以直接像指针那样解引用： *result
        std::cout << "找到了: " << result.value() << "\n";
    } else {
        std::cout << "抱歉，查无此人。\n";
    }

    // 神级函数：value_or() —— 取出来，如果是空的就给个默认保底值！
    std::cout << "VIP嘉宾是: " << findUser(5).value_or("Anonymous") << "\n";
}
```

## std::variant：类型安全的终极联合体（Union）

在 C 语言中如果一个变量可能是整数也可能是浮点，你会使用 `union`。但传统的 `union` 极度不安全，它不知道当前自己到底存的哪一种，乱读内存会导致灾难，并且它无法存复杂的拥有构造函数的类（比如 `std::string`）。

`std::variant<T1, T2, ...>` 是类型安全的。它在任何时刻**一定只存有一种明确指出的类型**。

```cpp
#include <iostream>
#include <variant>
#include <string>

int main() {
    // 这个包里可以放 int 或者 double 或者 string 的其中一个
    std::variant<int, double, std::string> v;

    v = 42; // 现在它是 int
    v = 3.14; // 现在切换形态成了 double
    v = "Hello variant"; // 最终成了字符串

    // 1. 获取值的手段：std::get<T>
    // 必须要类型完全匹配才能拿出来，否则抛出 std::bad_variant_access 异常
    std::cout << std::get<std::string>(v) << "\n";

    // 2. 探针手段：std::holds_alternative
    if (std::holds_alternative<int>(v)) {
        std::cout << "现在里边装的是个整数！\n";
    }
}
```
*(注：处理 variant 最推崇使用的是极其花哨但也稍显复杂的 `std::visit`，使用变参泛型手段去巡场，这里出于篇幅不做深入。)*

## std::any：我真不知道里面装什么

终极杂货筐，任何符合可拷贝特征的类都可以丢进里面存放，类型信息会在运行时被记录。
你可以把它视作 C 语言里面万能的 `void*` 的类型安全现代进化版（或者 Java 里的 `Object`）。

```cpp
#include <iostream>
#include <any>
#include <string>
#include <vector>

int main() {
    std::any bucket; // 就这么狂，完全不需要告诉它要装什么大类的类型

    bucket = 10;
    bucket = std::string("wow");
    bucket = std::vector<int>{1, 2, 3};

    // 要想拿出来用，必须使用 std::any_cast 做显式强投
    try {
        auto vec = std::any_cast<std::vector<int>>(bucket);
        std::cout << "拿出了包含 " << vec.size() << " 个元素的数组。\n";

        // 下面这句如果盲目转，将在运行时因为类型不符惨遭 std::bad_any_cast 抛库异常报错
        // int wrong_cast = std::any_cast<int>(bucket);
    } catch(const std::bad_any_cast& e) {
        std::cout << "转型失败啦！\n";
    }
}
```

> **最佳实践提醒：**
> 日常业务请高频使用 `std::optional`，在极其确定的多种选项汇流时才使用 `std::variant`。
> 至于 `std::any`？尽量远离除非你在写极其底层的框架反射、跨语言桥接或者事件总线监听这类不可预知的泥潭——因为泛型本来是为了在**编译期**确定雷区，使用了 `any` 等于你亲手把安全线拖到了**运行时（Runtime）**靠人品和异常监控去维护。