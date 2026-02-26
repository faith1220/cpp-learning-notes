# 9.4 const 与指针

## 指针与 const 的组合

指针和 `const` 的组合有多种形式，初学者容易混淆。关键在于 `const` 修饰的是什么——是指针指向的数据，还是指针本身，还是两者都有。

## 指向常量的指针

**指向常量的指针**（Pointer to const）：不能通过指针修改所指向的值，但指针本身可以指向其他对象。

```cpp
int x = 10;
const int* ptr = &x;    // ptr 是指向 const int 的指针

// *ptr = 20;           // 编译错误！不能通过 ptr 修改 x
x = 20;                 // 正确：直接修改 x 本身是允许的

int y = 30;
ptr = &y;               // 正确：ptr 可以指向其他对象
```

`const int*` 读作「指向 `const int` 的指针」。注意，这并不意味着被指向的变量一定是常量——只是约定「不通过这个指针修改它」。

### 等价写法

以下两种写法完全等价：

```cpp
const int* ptr1 = &x;    // const 在前
int const* ptr2 = &x;    // const 在后，意义相同
```

两者都表示「指向常量的指针」。推荐使用第一种（`const int*`），因为更符合自然语序。

## 常量指针

**常量指针**（Const pointer）：指针本身是常量，不能指向其他对象，但可以通过指针修改所指向的值。

```cpp
int x = 10;
int* const ptr = &x;    // ptr 是一个常量指针

*ptr = 20;              // 正确：可以修改 x 的值
std::cout << x << std::endl;   // 20

int y = 30;
// ptr = &y;            // 编译错误！ptr 是常量，不能重新指向
```

`int* const` 读作「指向 `int` 的常量指针」。`const` 在 `*` 之后，修饰的是指针变量 `ptr` 本身。

## 指向常量的常量指针

结合两者，既不能修改指向的值，也不能改变指针的指向：

```cpp
int x = 10;
const int* const ptr = &x;   // 指向 const int 的常量指针

// *ptr = 20;                // 错误：不能修改所指向的值
// ptr = &y;                 // 错误：不能改变指针的指向
```

`const int* const` 读作「指向 `const int` 的常量指针」。两个 `const` 分别约束指向的数据和指针本身。

## 记忆技巧：从右往左读

C++ 的 const 声明遵循「右结合」规则，从右往左读容易理解：

```cpp
const int* ptr;         // ptr 是指针，指向 const int
int* const ptr;         // ptr 是 const 指针，指向 int
const int* const ptr;   // ptr 是 const 指针，指向 const int
```

更系统的方法：**`const` 修饰它左边的东西**，如果左边没有东西，就修饰右边：

```cpp
const int* ptr;         // const 左边没有东西，修饰 int
int const* ptr;         // const 修饰 int
int* const ptr;         // const 修饰 *（即指针本身）
const int* const ptr;   // 两个 const，分别修饰 int 和 *
```

## 对比总结表

| 声明 | 能否修改指向的值 | 能否改变指针指向 | 读法 |
|------|----------------|----------------|------|
| `int* ptr` | ✓ | ✓ | 普通指针 |
| `const int* ptr` | ✗ | ✓ | 指向常量的指针 |
| `int* const ptr` | ✓ | ✗ | 常量指针 |
| `const int* const ptr` | ✗ | ✗ | 指向常量的常量指针 |

## 顶层 const 与底层 const

这两个术语帮助区分 const 的作用对象：

- **顶层 const**（Top-level const）：修饰变量本身是常量
- **底层 const**（Low-level const）：修饰指针所指对象是常量

```cpp
int x = 10;

const int* ptr1 = &x;         // 底层 const：所指对象是常量
int* const ptr2 = &x;         // 顶层 const：指针本身是常量
const int* const ptr3 = &x;   // 既有顶层也有底层 const
```

### 赋值兼容性

**底层 const 在拷贝时有严格限制**：不能将底层 const 对象赋给非 const：

```cpp
int x = 10;
const int* p1 = &x;      // p1 有底层 const
// int* p2 = p1;         // 错误！丢失底层 const

const int* p3 = p1;      // 正确：保留底层 const
```

**顶层 const 在拷贝时会被忽略**：

```cpp
int* const p1 = &x;      // p1 有顶层 const
int* p2 = p1;            // 正确：拷贝时忽略顶层 const
```

## 指针参数与 const

### 常见的参数传递模式

```cpp
// 1. 值传递：拷贝参数，函数内修改不影响外部
void foo(int x);

// 2. 指针传递：可修改外部变量
void bar(int* ptr);

// 3. 指向常量的指针：不可修改外部变量
void baz(const int* ptr);

// 4. 引用传递：可修改外部变量
void qux(int& ref);

// 5. const 引用传递：不可修改外部变量（推荐用于大对象）
void quux(const int& ref);
```

### 设计原则

- **传入不修改的大对象**：用 `const T&` 或 `const T*`（引用更简洁）
- **传入需修改的对象**：用 `T&` 或 `T*`（引用优先，指针用于可空情况）
- **传入小对象或基本类型**：直接值传递

```cpp
// 好的设计
void process(const std::string& s) {    // const 引用：大对象只读
    std::cout << s << std::endl;
}

void increment(int& n) {                 // 引用：小对象修改
    n++;
}

void findMax(const int* arr, int size) { // const 指针：数组只读
    // ...
}
```

## 数组与 const

数组名退化为指针时，const 也会体现在指针类型上：

```cpp
int arr[5] = {1, 2, 3, 4, 5};
const int carr[5] = {1, 2, 3, 4, 5};

int* p1 = arr;           // 正确
// int* p2 = carr;       // 错误！carr 是 const int[]，退化为 const int*

const int* p3 = carr;    // 正确
const int* p4 = arr;     // 正确：普通数组可以赋给指向常量的指针
```

传递数组给函数时，如果不修改数组，应声明为 `const`：

```cpp
int sum(const int* arr, int size) {
    int total = 0;
    for (int i = 0; i < size; ++i) {
        total += arr[i];
        // arr[i] = 0;    // 错误！const 不允许修改
    }
    return total;
}
```

## 常见问题

**Q：`const int*` 和 `int const*` 真的完全一样吗？**

是的，完全等价。但约定俗成地，大多数代码使用 `const int*` 的写法。

**Q：为什么 const 引用比指向常量的指针更常用？**

引用语法更简洁（不需要 `*` 和 `&`），且不存在空值问题。对于参数传递，`const T&` 几乎总是比 `const T*` 更好的选择，除非需要表示「可选参数」（可为空）的语义。

**Q：`const int* ptr` 指向的数据真的不能修改吗？**

通过这个指针不能修改，但如果有其他途径访问该数据（如原变量或其他非 const 指针），仍然可以修改。`const` 只是一种编译期约定，不是运行时保护。
