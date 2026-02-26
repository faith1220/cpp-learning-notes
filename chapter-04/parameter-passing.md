# 4.2 参数传递

函数的参数传递方式决定了函数内部对参数的操作是否会影响到调用者。C++ 支持**值传递**和**引用传递**两种基本方式。

## 值传递

默认情况下，C++ 使用**值传递**（Pass by Value）。调用函数时，实际参数的值被**复制**给形式参数。函数内部对形式参数的修改不会影响原始变量。

```cpp
void doubleValue(int x) {
    x = x * 2;   // 修改的是副本，不影响外部变量
}

int main() {
    int num = 10;
    doubleValue(num);
    std::cout << num << std::endl;   // 输出 10，不是 20
    return 0;
}
```

值传递的优点是安全——函数不会意外修改调用者的数据。缺点是对于大型对象（如长字符串、大数组），复制操作会带来额外开销。

## 引用传递

**引用传递**（Pass by Reference）通过在参数类型后加 `&` 来声明。此时形式参数是实际参数的**别名**，函数内部的修改会直接作用于原始变量。

```cpp
void doubleValue(int& x) {   // x 是引用参数
    x = x * 2;   // 直接修改原始变量
}

int main() {
    int num = 10;
    doubleValue(num);
    std::cout << num << std::endl;   // 输出 20
    return 0;
}
```

引用传递的典型用途：

1. **需要修改调用者的变量** — 如上例所示
2. **避免大对象的复制开销** — 传递大型结构体、容器等

### 交换两个变量的值

经典的引用传递示例——交换函数：

```cpp
void swap(int& a, int& b) {
    int temp = a;
    a = b;
    b = temp;
}

int main() {
    int x = 3, y = 7;
    swap(x, y);
    std::cout << "x=" << x << " y=" << y << std::endl;
    // 输出：x=7 y=3
    return 0;
}
```

如果使用值传递，`swap` 函数内部交换的只是副本，原始变量不受影响。

## const 引用

当函数只需要读取参数而不修改它时，使用 `const` 引用既能避免复制开销，又能防止意外修改：

```cpp
void print(const std::string& text) {
    std::cout << text << std::endl;
    // text = "其他内容";   // 编译错误：不能修改 const 引用
}
```

`const` 引用是传递大型只读参数的最佳实践：

| 参数类型 | 推荐传递方式 |
|----------|--------------|
| 内置类型（`int`、`double` 等） | 值传递 |
| 大型对象（`std::string`、容器等），只读 | `const` 引用 |
| 需要修改调用者的变量 | 引用 |

## 返回多个值

C++ 的函数只能有一个返回值。如果需要返回多个值，有几种常用方法。

### 方法一：引用参数

通过引用参数「带出」额外的返回值：

```cpp
void divide(int dividend, int divisor, int& quotient, int& remainder) {
    quotient = dividend / divisor;
    remainder = dividend % divisor;
}

int main() {
    int q, r;
    divide(17, 5, q, r);
    std::cout << "商：" << q << " 余数：" << r << std::endl;
    // 输出：商：3 余数：2
    return 0;
}
```

### 方法二：返回结构体或 std::pair

更现代的做法是返回一个包含多个值的结构体，或使用标准库的 `std::pair`：

```cpp
#include <utility>   // std::pair

std::pair<int, int> divide(int dividend, int divisor) {
    return {dividend / divisor, dividend % divisor};
}

int main() {
    auto [q, r] = divide(17, 5);   // C++17 结构化绑定
    std::cout << "商：" << q << " 余数：" << r << std::endl;
    return 0;
}
```

这种方式更加清晰，后续章节会进一步介绍结构化绑定和 `std::tuple`。

## 默认参数

函数的参数可以指定**默认值**（Default Argument）。调用时如果省略了该参数，就使用默认值：

```cpp
void printMessage(const std::string& msg, int times = 1) {
    for (int i = 0; i < times; ++i) {
        std::cout << msg << std::endl;
    }
}

int main() {
    printMessage("你好");        // times 使用默认值 1
    printMessage("你好", 3);     // times 为 3，输出三次
    return 0;
}
```

默认参数的规则：

- 默认参数必须从参数列表的**右侧**开始，不能跳过中间的参数
- 如果函数有声明和定义分离，默认值只能写在**声明**中，不能写在定义中

```cpp
// 合法
void func(int a, int b = 10, int c = 20);

// 不合法：默认参数不能跳过中间的 b
void func(int a, int b, int c = 20, int d);   // 错误
```
