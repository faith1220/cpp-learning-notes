# 14.1 auto 与 decltype

类型推导（Type Deduction）是一项极具生产力的特性。它让程序员专注于意图而不是繁杂的类型声明体系。

## auto 关键字（C++11）

`auto` 不是一种新的数据类型！它是一个占位符，告诉编译器：“**请根据我右边等号赋的值，自动推断并填上正确的类型**”。

### 基本用法

```cpp
auto i = 42;          // 被推测为 int
auto d = 3.14;        // 被推测为 double
auto s = "Hello";     // 被推测为 const char*
```

上面的例子可能看不出啥优势，甚至觉得降低了可读性。`auto` 的真正威力在于配合**迭代器**和**复杂模板返回类型**使用：

```cpp
#include <vector>
#include <string>
#include <map>

int main() {
    std::map<std::string, std::vector<int>> complexMap;

    // C++98 噩梦般的写法
    for (std::map<std::string, std::vector<int>>::const_iterator it = complexMap.begin(); it != complexMap.end(); ++it) {
        // ...
    }

    // C++11 现代写法
    for (auto it = complexMap.begin(); it != complexMap.end(); ++it) {
        // ...
    }
}
```

### auto 的推导规则（去 const 和引用）

这是一个容易被忽视的细节：当使用纯粹的 `auto` 时，编译器**默认会剥离掉右值表达式的顶层 `const` 属性和引用（`&`）属性**。这就会表现为“值拷贝”。

```cpp
const int cx = 10;
const int& rx = cx;

auto a = rx; // a 的类型会被推导为普通的 int！它仅仅是 rx 的一个【拷贝副本】。
a = 20;      // 完全合法，改的是副本 a
```

如果你的本意是不想拷贝，而是继续保持引用或 `const` 约束，你需要手动把饰辞给 `auto` 加上：

```cpp
const auto& b = rx; // b 推导为 const int& 表明它只是个只读引用
auto& c = rx;       // 因为 rx 底层带有 const 限制没法变异，所以 c 被推断为 const int&
```

## decltype 关键字（C++11）

`decltype` 的发音类似于 "declare type"。它的作用是：**“编译器，请告诉我括号里那个表达式的准确类型，并把那个类型拿出来给我用！”**

与 `auto` 必须要就地赋值初始化不同，`decltype` 只是提取类型，并不需要计算表达式的值。这允许我们仅仅把提取出的类型用于变量声明或者函数返回类型。

```cpp
int x = 0;
double y = 0.0;

// 推导 x + y 运算后的类型（显然整型+浮点型=浮点型 double）
decltype(x + y) z; // z 的类型就是 double，此时 z 没有初始化
z = 3.14;
```

### decltype 与引用的保留

与 `auto` 疯狂丢弃引用不同，**`decltype` 会原汁原味地保留表达式的一切属性（包括引用和 const）**。

```cpp
const int cx = 10;
const int& rx = cx;

decltype(cx) a = 20; // a 就是 const int，所以这行不仅被推导，连赋值也得就地写，之后不准改。
decltype(rx) b = a;  // b 的完全类型被推导为 const int& (原汁原味)
```

### 结合 auto 返回类型后置（Trailing Return Type）

在模版编程中，如果在函数定义时你还不知道返回值是什么（因为它取决于送进来的模板参数运算后决定），可以用 `auto` 和 `decltype` 绝妙配合：

```cpp
template<typename T1, typename T2>
// 我们在参数列表解析完毕后，才知道此时把 a 和 b 加起来是什么类型
// -> decltype(...) 就是告诉世界：我的返回类型就是括号里加完的结果类型！
auto add(T1 a, T2 b) -> decltype(a + b) {
    return a + b;
}

// （注：在 C++14 后，普通的 auto 也可以直接做返回类型而不用后缀箭头推导了）
template<typename T1, typename T2>
auto add_cpp14(T1 a, T2 b) {
    return a + b;
}
```