# 15.3 if constexpr

我们在第 11 章学习过【模板】，模板的逻辑是在“编译期（Compile Time）”做代码图纸的推演生成的。
而在第 3 章学过的普通的 `if...else...` 是在“运行时（Runtime）”去判明并决定执行哪一段二进制代码分支的。

如果在一个模板函数里，我们通过判断送进来的某个类的类型特征，试图走不同的普通 `if` 分支。由于普通的 `if` 两个分支在编译期都必须被成功生成机器码，遇到和某一边分支类型不和的操作往往就会触发直接的编译大报错！

C++17 引入了革命性的 **`if constexpr`**（编译期条件分支），它允许我们在**编译期计算条件，然后直接丢弃不需要的那个死胡同分支（它连编译都不会去编译那一端！）**。

## 对比痛点与解法

假设我们要写一个泛型函数，如果是**指针类型**才去解引用提取它的值，遇到普通的变量就直接返回值本身。

```cpp
#include <iostream>
#include <type_traits> // 提供大量判断类型的萃取器

// 传统做法报错演示（由于我们无法编译演示这就注释掉了）
/*
template <typename T>
auto getValueFail(T val) {
    // 假设外界送了个普通的 int 5 进来。
    // 虽然运行逻辑上，指针的判定肯定是假，他会走直接 return 5 的 else
    if (std::is_pointer<T>::value) {
        return *val; // 💥 但编译器很死脑筋，因为上面这个代码块它也要翻译！编译器怒吼：你要对一个非指针类型 int 5 解引用？！编译报错炸穿！
    } else {
        return val;
    }
}
*/

// C++17 魔法解决
template <typename T>
auto getValue(T val) {
    // 加上 constexpr 后。如果 T 传来的是 int，编译器会直接【物理抹除删除】第一个 if 块及其里面的代码。
    // 就当作没看见！
    if constexpr (std::is_pointer<T>::value) {
        return *val;
    } else {
        return val;
    }
}

int main() {
    int   a = 42;
    int* pa = &a;

    std::cout << getValue(a) << "\n";  // 输出 42 (安全过滤了 *val 的操作)
    std::cout << getValue(pa) << "\n"; // 输出 42
}
```

## 为什么说它是天降神兵？

在 `if constexpr` 出现之前，要想做类似上面那种针对某些特定类型走特殊逻辑的操作：

C++ 程序员不得不疯狂手写无数遍**模板的特化 / 偏特化**方案又或者是利用古老的 SFINAE/`enable_if` 奇技淫巧去强行在外部切断重载函数的匹配路径。这带来了极其恐怖的模板代码膨胀。

现在，我们可以把针对五湖四海各式类型处理流向分支，用简单直观如同凡人说话一般的 `if...else if...else` 结构全部装在**同一个函数里**了！只要条件是能在编译期通过 `type_traits` 工具确定的属性就行。