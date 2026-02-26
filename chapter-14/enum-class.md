# 14.5 强类型枚举类（enum class）

传统的 C 风格枚举（Enums）在现代 C++ 程序员眼里被叫做**“不限定作用域枚举（unscoped enums）”**。它存在两大被诟病已久的顽疾：**命名空间的污染** 和 **不安全的隐式转换为整数**。

为了解决这两个痛点，C++11 引入了更加现代安全的 **枚举类（enum class）**（或者叫做强类型枚举 strongly typed enums）。

## 痛点 1：命名污染

以前如果要定两个传统的枚举来代表文件打开方式和机器的状态：

```cpp
enum FileAccess { Read, Write, Append };
enum ProcessStatus { Init, Write, Finish }; // 错误！编译不通过！
```
尽管在不同的 enum 声明里，传统方法下的所有的枚举项（如 `Read`，`Write`）直接等同于全部泄漏到外层的全局作用域中去。你不能在同域的不同枚举中享有相同的字眼！

### 解决方案：enum class
给 `enum` 后面加上 `class` 关键字声明强类型枚举。此时的各个定义域互不侵犯，外界如果想访问具体的枚举子，必须显式加上枚举类型的名称+双冒号作用域符 `::`。

```cpp
enum class FileAccess { Read, Write, Append };
enum class ProcessStatus { Init, Write, Finish }; // 没问题了

// 使用时：
FileAccess myMode = FileAccess::Write;
```

## 痛点 2：极其随便地与整数混杂（隐式转换）

在底层，C 语言的做法就是直接给枚举每一个值编了个整数 0, 1, 2。所以它们可以被非常危险地视同整型来随时运算，编译器都不会阻拦！

```cpp
enum Color { Red, Green, Blue };
Color myFav = Red;

if (myFav < 1.5) { // 居然可以用小数跟颜色比较大小？！合法且荒谬。
    // ...
}
```

### 解决方案：enum class 严格把关
强类型枚举不仅限定了命名墙，它具有真正**强类型**（Strongly Typed）的数据地位：它再也不允许被隐式发生和整数与浮点的降级转换！

```cpp
enum class Color { Red, Green, Blue };
Color myFav = Color::Red;

// if (myFav < 1.5) // 编译报错！类型不允许。

// 除非你强行做 C++ 风格显式类型强制转换（static_cast）告知这事由你负责：
if (static_cast<int>(myFav) < 1) {
    // ...
}
```

## 前置声明与指定底层类型

传统 C 枚举只有在编译器看完里面的所有元素才能计算出这个枚举要占用几个字节。这就导致传统的 C 风格枚举不能做前向声明（Forward Declaration）。

C++11 允许在声明任意枚举时显式指定底层的整型存放尺寸：

```cpp
// 强烈暗示编译器：我就只需要用到无符号字符这 1 个字节的大小存它！
enum class SmallEnum : unsigned char {
    ItemA,
    ItemB
};
```
这样不仅极大节省内存打包空间，也因为确定的包体大小获得了被拆成头文件提前声明的能力。