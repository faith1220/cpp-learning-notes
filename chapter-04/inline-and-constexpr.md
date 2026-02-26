# 4.5 内联函数与 constexpr

## 函数调用的开销

每次调用函数时，系统需要执行一系列操作：保存当前执行位置、压入参数、跳转到函数代码、执行函数体、将返回值传回、恢复执行位置。这些操作称为**函数调用开销**（Function Call Overhead）。

对于大型函数，这些开销相对于函数体的执行时间可以忽略不计。但对于非常短小的函数（如只有一行的简单计算），调用开销可能占比较大。C++ 提供了两种机制来优化这种情况。

## 内联函数

**内联函数**（Inline Function）通过 `inline` 关键字提示编译器：在调用处直接展开函数体，而不是执行常规的函数调用。

```cpp
inline int square(int x) {
    return x * x;
}

int main() {
    int result = square(5);
    // 编译器可能将其展开为：int result = 5 * 5;
    return 0;
}
```

### 内联的注意事项

- `inline` 只是对编译器的**建议**，编译器可以忽略它。现代编译器会自行判断是否内联，即使没有 `inline` 关键字也可能对短小函数执行内联优化。
- 内联适合**函数体很短**（通常几行以内）的函数。对于复杂函数，内联反而可能增大生成的代码体积，降低缓存效率。
- 内联函数的定义通常放在**头文件**中，因为编译器在调用处需要看到函数体才能展开。

在实际开发中，你很少需要手动添加 `inline`。现代编译器的优化能力足以自行做出正确的内联决策。`inline` 关键字在实践中更多用于解决**头文件中函数定义的链接问题**，而非性能优化（这一点在第十八章讨论头文件组织时会进一步说明）。

## constexpr 函数

C++11 引入了 `constexpr` 函数，表示该函数**有可能在编译期求值**。如果调用时传入的参数都是编译期常量，编译器会在编译阶段直接计算出结果，不产生任何运行时开销。

```cpp
constexpr int square(int x) {
    return x * x;
}

int main() {
    constexpr int a = square(5);   // 编译期计算，a 直接被设为 25
    int b = 3;
    int c = square(b);             // b 不是编译期常量，运行时计算
    return 0;
}
```

### constexpr 函数的规则

在 C++11 中，`constexpr` 函数的限制较严格：函数体只能包含一条 `return` 语句。C++14 放宽了限制，允许使用局部变量、条件语句和循环：

```cpp
// C++14 起合法
constexpr int factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; ++i) {
        result *= i;
    }
    return result;
}

constexpr int f5 = factorial(5);   // 编译期计算，f5 为 120
```

`constexpr` 函数的关键特性：

- 如果参数是编译期常量，函数在编译期求值
- 如果参数是运行时值，函数退化为普通函数在运行时求值
- `constexpr` 函数隐含 `inline`

### constexpr 与 const 的区别

回顾一下二者的区别：

```cpp
const int a = 10;          // 运行时常量，值在运行期确定后不可修改
constexpr int b = 10;      // 编译期常量，值必须在编译期就能确定

const int c = getValue();      // 合法：const 允许运行期初始化
constexpr int d = getValue();  // 仅当 getValue() 是 constexpr 函数时才合法
```

## 实际应用

`constexpr` 函数常用于需要编译期常量的场景，例如数组大小：

```cpp
constexpr int arraySize(int rows, int cols) {
    return rows * cols;
}

int main() {
    int data[arraySize(3, 4)];   // 编译期确定数组大小为 12
    return 0;
}
```

如果用普通函数，数组大小无法在编译期确定，代码将无法编译（C++ 标准不支持变长数组）。

## 总结

| 特性 | `inline` | `constexpr` |
|------|----------|-------------|
| 作用 | 建议编译器在调用处展开函数体 | 允许函数在编译期求值 |
| 强制性 | 仅建议，编译器可忽略 | 满足条件时编译器必须在编译期求值 |
| 适用场景 | 短小的辅助函数 | 需要编译期常量的计算 |
| 是否隐含 inline | 否 | 是 |
