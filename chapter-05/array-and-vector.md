# 5.4 std::array 与 std::vector 初步

原生数组存在不知道自身大小、不支持赋值和比较、不做越界检查等问题。C++ 标准库提供了两个容器来替代原生数组：**`std::array`**（固定大小）和 **`std::vector`**（动态大小）。本节做初步介绍，更详细的讨论将在第十二章「STL 容器」中展开。

## std::array

`std::array` 定义在 `<array>` 头文件中，是对原生数组的**轻量封装**。它的大小在编译期确定，与原生数组一样存储在栈上，性能完全相同，但提供了更安全和方便的接口。

### 声明与初始化

```cpp
#include <array>

std::array<int, 5> scores = {90, 85, 78, 92, 88};
std::array<double, 3> values = {1.1, 2.2, 3.3};
std::array<int, 4> zeros = {};   // 全部初始化为 0
```

模板参数中，第一个是元素类型，第二个是数组大小（必须是编译期常量）。

### 基本操作

```cpp
std::array<int, 5> arr = {10, 20, 30, 40, 50};

// 获取大小
std::cout << arr.size() << std::endl;    // 5

// 访问元素
std::cout << arr[0] << std::endl;        // 10（不检查越界）
std::cout << arr.at(2) << std::endl;     // 30（越界时抛出异常）
std::cout << arr.front() << std::endl;   // 10
std::cout << arr.back() << std::endl;    // 50

// 修改元素
arr[0] = 100;
arr.at(1) = 200;
```

### 相比原生数组的优势

```cpp
std::array<int, 3> a = {1, 2, 3};
std::array<int, 3> b = {4, 5, 6};

// 支持直接赋值
b = a;   // b 变为 {1, 2, 3}

// 支持比较
if (a == b) {
    std::cout << "两个数组相等" << std::endl;
}

// 知道自身大小，传递给函数时不会丢失
void print(const std::array<int, 3>& arr) {
    for (const auto& elem : arr) {
        std::cout << elem << " ";
    }
}
```

### 何时使用 std::array

当数组大小在编译期已知且不需要改变时，使用 `std::array` 替代原生数组。它没有额外的运行时开销，却提供了更好的安全性和便利性。

## std::vector

`std::vector` 定义在 `<vector>` 头文件中，是 C++ 中最常用的容器。与 `std::array` 不同，`std::vector` 的大小可以在运行时**动态增长和缩减**。

### 声明与初始化

```cpp
#include <vector>

std::vector<int> v1;                    // 空 vector
std::vector<int> v2 = {1, 2, 3, 4, 5}; // 列表初始化
std::vector<int> v3(10);               // 10 个元素，全部初始化为 0
std::vector<int> v4(5, 42);            // 5 个元素，全部初始化为 42
std::vector<std::string> names = {"Alice", "Bob", "Carol"};
```

### 基本操作

```cpp
std::vector<int> v = {10, 20, 30};

// 获取大小
std::cout << v.size() << std::endl;     // 3
std::cout << v.empty() << std::endl;    // false

// 访问元素
std::cout << v[0] << std::endl;         // 10
std::cout << v.at(1) << std::endl;      // 20
std::cout << v.front() << std::endl;    // 10
std::cout << v.back() << std::endl;     // 30
```

### 动态增删元素

这是 `std::vector` 相比固定大小数组最大的优势：

```cpp
std::vector<int> v;

// 在末尾添加元素
v.push_back(10);    // v: {10}
v.push_back(20);    // v: {10, 20}
v.push_back(30);    // v: {10, 20, 30}

// 删除末尾元素
v.pop_back();        // v: {10, 20}

// 获取当前大小
std::cout << v.size() << std::endl;   // 2

// 清空所有元素
v.clear();           // v: {}
```

### 遍历

```cpp
std::vector<int> v = {1, 2, 3, 4, 5};

// 下标遍历
for (size_t i = 0; i < v.size(); ++i) {
    std::cout << v[i] << " ";
}

// 范围 for 循环（推荐）
for (int x : v) {
    std::cout << x << " ";
}

// const 引用遍历（避免拷贝）
for (const auto& x : v) {
    std::cout << x << " ";
}
```

### 赋值与比较

```cpp
std::vector<int> a = {1, 2, 3};
std::vector<int> b = {4, 5, 6};

b = a;               // 深拷贝，b 变为 {1, 2, 3}

if (a == b) {
    std::cout << "相等" << std::endl;
}
```

### 作为函数参数

`std::vector` 作为函数参数时，推荐使用 `const` 引用以避免不必要的拷贝：

```cpp
double average(const std::vector<int>& v) {
    if (v.empty()) return 0.0;
    int sum = 0;
    for (int x : v) {
        sum += x;
    }
    return static_cast<double>(sum) / v.size();
}

int main() {
    std::vector<int> scores = {90, 85, 78, 92, 88};
    std::cout << "平均分：" << average(scores) << std::endl;
    return 0;
}
```

函数内部可以通过 `v.size()` 获取元素数量，不再需要额外传递大小参数。

## 三者对比

| 特性 | 原生数组 | `std::array` | `std::vector` |
|------|----------|--------------|---------------|
| 大小 | 编译期固定 | 编译期固定 | 运行时可变 |
| 知道自身大小 | 否 | 是 | 是 |
| 越界检查 | 无 | `.at()` 提供 | `.at()` 提供 |
| 支持赋值 `=` | 否 | 是 | 是 |
| 支持比较 `==` | 否 | 是 | 是 |
| 存储位置 | 栈 | 栈 | 堆（自动管理） |
| 额外开销 | 无 | 无 | 极小（容量管理） |
| 头文件 | 无需 | `<array>` | `<vector>` |

{% hint style="info" %}
在现代 C++ 中，推荐的做法是：大小固定时使用 `std::array`，大小可变时使用 `std::vector`。除非有特殊理由（如与 C 代码交互），否则不要使用原生数组。
{% endhint %}
