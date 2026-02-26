# 5.1 数组

## 什么是数组

**数组**（Array）是一组**相同类型**的元素在内存中**连续存储**的数据结构。每个元素通过**下标**（Index）来访问，下标从 0 开始。

## 声明与初始化

数组的声明语法：

```cpp
类型 数组名[大小];
```

```cpp
int scores[5];          // 声明一个包含 5 个 int 的数组（未初始化）
double prices[10];      // 声明一个包含 10 个 double 的数组
```

声明时可以同时初始化：

```cpp
int scores[5] = {90, 85, 78, 92, 88};    // 完整初始化
int values[5] = {1, 2, 3};               // 部分初始化，剩余元素为 0
int zeros[5] = {};                        // 全部初始化为 0
int auto_size[] = {10, 20, 30};           // 编译器根据初始值推断大小为 3
```

{% hint style="warning" %}
未初始化的局部数组中的元素值是不确定的。始终在声明时初始化数组，至少使用 `= {}` 将所有元素置零。
{% endhint %}

数组的大小必须是**编译期常量**，不能是运行时变量：

```cpp
const int SIZE = 5;
int arr1[SIZE];          // 合法：SIZE 是编译期常量

int n = 5;
int arr2[n];             // 不合法（标准 C++ 不支持变长数组）
```

## 访问元素

通过下标运算符 `[]` 访问数组元素，下标从 0 开始：

```cpp
int scores[5] = {90, 85, 78, 92, 88};

std::cout << scores[0] << std::endl;   // 输出 90（第一个元素）
std::cout << scores[4] << std::endl;   // 输出 88（最后一个元素）

scores[2] = 100;                       // 修改第三个元素
```

{% hint style="danger" %}
C++ 不会检查数组下标是否越界。访问 `scores[5]` 或 `scores[-1]` 不会报编译错误，但会导致**未定义行为**（Undefined Behavior），可能读写到不属于数组的内存区域，引发难以排查的 bug 甚至程序崩溃。始终确保下标在 `[0, 大小-1]` 范围内。
{% endhint %}

## 遍历数组

### 使用 for 循环

```cpp
int scores[5] = {90, 85, 78, 92, 88};

for (int i = 0; i < 5; ++i) {
    std::cout << "scores[" << i << "] = " << scores[i] << std::endl;
}
```

### 使用范围 for 循环

```cpp
for (int s : scores) {
    std::cout << s << " ";
}
std::cout << std::endl;
```

范围 for 循环更安全，不存在下标越界的风险。

## 数组作为函数参数

将数组传递给函数时，数组会退化为指向首元素的**指针**（Pointer），函数内部无法知道数组的大小。因此通常需要额外传递一个表示大小的参数：

```cpp
double average(int arr[], int size) {
    int sum = 0;
    for (int i = 0; i < size; ++i) {
        sum += arr[i];
    }
    return static_cast<double>(sum) / size;
}

int main() {
    int scores[] = {90, 85, 78, 92, 88};
    std::cout << "平均分：" << average(scores, 5) << std::endl;
    return 0;
}
```

数组参数本质上是指针传递，因此函数内部对数组元素的修改会影响原始数组。

## 多维数组

数组可以嵌套，形成**多维数组**。最常用的是二维数组，可以用来表示矩阵或表格：

```cpp
int matrix[3][4] = {
    {1, 2, 3, 4},
    {5, 6, 7, 8},
    {9, 10, 11, 12}
};

// 访问第 2 行第 3 列（下标从 0 开始）
std::cout << matrix[1][2] << std::endl;   // 输出 7
```

遍历二维数组需要嵌套循环：

```cpp
for (int i = 0; i < 3; ++i) {
    for (int j = 0; j < 4; ++j) {
        std::cout << matrix[i][j] << "\t";
    }
    std::cout << std::endl;
}
```

## 原生数组的局限性

C++ 原生数组存在若干不便之处：

- **不知道自身大小** — 传递给函数后丢失长度信息
- **不支持直接赋值和比较** — 不能用 `=` 复制数组，也不能用 `==` 比较两个数组
- **不做越界检查** — 越界访问是未定义行为
- **大小固定** — 声明后无法改变容量

这些问题促使 C++ 标准库提供了更安全、更灵活的替代方案：`std::array` 和 `std::vector`，将在本章最后一节介绍。
