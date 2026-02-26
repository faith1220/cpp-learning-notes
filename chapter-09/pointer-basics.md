# 9.1 指针基础

## 内存与地址

程序运行时，每个变量都存储在内存中的某个位置。内存可以想象为一排编了号的格子，每个格子有一个唯一的编号，称为**地址**（Address）。

```cpp
int x = 42;
std::cout << "x 的值：" << x << std::endl;
std::cout << "x 的地址：" << &x << std::endl;  // 输出类似 0x7ffd5a3c
```

`&` 是**取地址运算符**（Address-of Operator），`&x` 获取变量 `x` 在内存中的地址。地址通常以十六进制显示。

## 什么是指针

**指针**（Pointer）是一个变量，它存储的不是普通数据，而是另一个变量的**地址**。

```cpp
int x = 42;
int* ptr = &x;    // ptr 是指向 int 的指针，存储了 x 的地址
```

声明语法：`类型* 变量名`。`int*` 表示「指向 int 的指针」。`*` 靠近类型名或变量名都可以，以下写法等价：

```cpp
int* ptr;    // 推荐：强调 ptr 的类型是 int*
int *ptr;    // C 风格：强调 *ptr 是 int
int * ptr;   // 也合法，但较少使用
```

{% hint style="warning" %}
在同一行声明多个变量时，`*` 只绑定到紧跟的变量名：`int* a, b;` 中 `a` 是指针，`b` 是普通 `int`。建议每行只声明一个指针变量。
{% endhint %}

## 解引用

通过指针访问它所指向的变量，使用 `*`（**解引用运算符**，Dereference Operator）：

```cpp
int x = 42;
int* ptr = &x;

std::cout << ptr << std::endl;    // 输出地址，如 0x7ffd5a3c
std::cout << *ptr << std::endl;   // 输出 42（ptr 指向的值）

*ptr = 100;                       // 通过指针修改 x 的值
std::cout << x << std::endl;      // 100
```

`*ptr` 和 `x` 指代同一块内存。修改 `*ptr` 就是修改 `x`。

```
变量 x：  [ 42 ]     地址：0x1000
指针 ptr：[ 0x1000 ]  地址：0x2000
           ↓
         指向 x
```

## 指针的大小

指针本身也是一个变量，占用固定大小的内存。在 64 位系统上，所有类型的指针都是 **8 字节**；在 32 位系统上是 4 字节：

```cpp
std::cout << sizeof(int*) << std::endl;       // 8（64 位系统）
std::cout << sizeof(double*) << std::endl;    // 8
std::cout << sizeof(char*) << std::endl;      // 8
```

指针的大小取决于系统的地址位宽，与它指向的数据类型无关。但指针的类型决定了解引用时如何解读内存中的数据。

## 空指针

不指向任何对象的指针称为**空指针**（Null Pointer）。C++11 引入了 `nullptr` 关键字来表示空指针：

```cpp
int* ptr = nullptr;    // 推荐：C++11 空指针

if (ptr != nullptr) {
    std::cout << *ptr << std::endl;
} else {
    std::cout << "指针为空" << std::endl;
}
```

{% hint style="danger" %}
**解引用空指针是未定义行为**，通常导致程序崩溃（段错误）。使用指针前务必检查是否为空。
{% endhint %}

在 C++11 之前，空指针用 `NULL`（宏定义，通常是 `0`）或字面量 `0` 表示。`nullptr` 比它们更安全，因为 `nullptr` 的类型是 `std::nullptr_t`，不会与整数混淆：

```cpp
void foo(int n) { std::cout << "int" << std::endl; }
void foo(int* p) { std::cout << "int*" << std::endl; }

foo(0);          // 调用 foo(int)——不是你想要的
foo(NULL);       // 可能有二义性（取决于 NULL 的定义）
foo(nullptr);    // 明确调用 foo(int*)
```

## 指针与数组

数组名在大多数表达式中会**退化**（Decay）为指向首元素的指针：

```cpp
int arr[5] = {10, 20, 30, 40, 50};
int* ptr = arr;        // arr 退化为 &arr[0]

std::cout << *ptr << std::endl;        // 10（首元素）
std::cout << *(ptr + 1) << std::endl;  // 20（第二个元素）
std::cout << *(ptr + 2) << std::endl;  // 30（第三个元素）
```

### 指针算术

对指针进行加减运算时，单位不是字节，而是指针**所指类型的大小**：

```cpp
int arr[5] = {10, 20, 30, 40, 50};
int* ptr = arr;

ptr + 1;    // 地址增加 sizeof(int) = 4 字节，指向 arr[1]
ptr + 3;    // 地址增加 12 字节，指向 arr[3]
```

因此，`*(ptr + i)` 等价于 `arr[i]`，实际上 `arr[i]` 在底层就是 `*(arr + i)` 的语法糖。

### 用指针遍历数组

```cpp
int arr[5] = {10, 20, 30, 40, 50};

for (int* p = arr; p < arr + 5; ++p) {
    std::cout << *p << " ";
}
// 输出：10 20 30 40 50
```

`arr + 5` 是**尾后指针**（Past-the-End Pointer），指向数组最后一个元素之后的位置。可以用于比较，但不能解引用。这个概念与 STL 迭代器的 `end()` 相同。

## 指针与函数

### 通过指针修改外部变量

```cpp
void doubleValue(int* ptr) {
    *ptr *= 2;
}

int main() {
    int x = 5;
    doubleValue(&x);
    std::cout << x << std::endl;   // 10
    return 0;
}
```

### 函数返回指针

```cpp
// 返回数组中最大元素的指针
int* findMax(int* arr, int size) {
    int* maxPtr = arr;
    for (int i = 1; i < size; ++i) {
        if (arr[i] > *maxPtr) {
            maxPtr = &arr[i];
        }
    }
    return maxPtr;
}

int main() {
    int arr[] = {3, 7, 2, 9, 5};
    int* p = findMax(arr, 5);
    std::cout << "最大值：" << *p << std::endl;   // 9
    return 0;
}
```

{% hint style="danger" %}
**永远不要返回指向局部变量的指针。** 函数返回后局部变量被销毁，指针变成**悬垂指针**（Dangling Pointer），解引用会导致未定义行为。
{% endhint %}

```cpp
int* bad() {
    int x = 42;
    return &x;     // 错误！x 在函数返回后被销毁
}
```

## 常见问题

**Q：指针和地址是一回事吗？**

地址是内存位置的编号（一个数值），指针是存储地址的变量。地址是值，指针是变量。类比：「北京」是一个地点，「写着北京的路牌」是指针。

**Q：`*` 在不同位置的含义？**

- 声明时：`int* ptr` — 定义一个指针变量
- 表达式中：`*ptr` — 解引用，访问指针指向的值
- 两者语法相同但含义不同，靠上下文区分

**Q：为什么需要指针？引用不就够了吗？**

指针比引用更灵活：指针可以为空、可以重新指向其他对象、可以进行算术运算、可以用于动态内存管理。引用在很多场景下更安全更方便，但无法完全取代指针。后续章节会详细讨论两者的适用场景。
