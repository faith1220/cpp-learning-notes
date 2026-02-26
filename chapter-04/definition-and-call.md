# 4.1 函数的定义与调用

## 函数的基本结构

一个函数由四部分组成：**返回类型**、**函数名**、**参数列表**和**函数体**。

```cpp
返回类型 函数名(参数列表) {
    函数体
    return 返回值;
}
```

示例——计算两个整数的最大值：

```cpp
int max(int a, int b) {
    if (a > b) {
        return a;
    } else {
        return b;
    }
}
```

- `int` 是返回类型，表示函数返回一个整数
- `max` 是函数名
- `(int a, int b)` 是参数列表，声明了两个整型参数
- `return` 语句将结果返回给调用者

## 调用函数

定义好的函数通过函数名加括号来调用，括号中传入**实际参数**（Argument）：

```cpp
#include <iostream>

int max(int a, int b) {
    return (a > b) ? a : b;
}

int main() {
    int result = max(10, 20);
    std::cout << "较大的值是 " << result << std::endl;

    // 函数调用也可以直接用在表达式中
    std::cout << "三个数的最大值：" << max(max(3, 7), 5) << std::endl;
    return 0;
}
```

调用函数时，实际参数的值会被复制给函数定义中的**形式参数**（Parameter）。函数执行完毕后，返回值会替换调用处的函数表达式。

## 无返回值函数

如果函数不需要返回值，返回类型声明为 `void`：

```cpp
void greet(std::string name) {
    std::cout << "你好，" << name << "！" << std::endl;
    // void 函数可以省略 return，或写 return; 不带值
}

int main() {
    greet("张三");
    return 0;
}
```

## 函数声明（前向声明）

C++ 要求函数在被调用之前必须是已知的。如果函数定义写在调用点之后，需要在调用之前提供**函数声明**（Function Declaration），也称为**函数原型**（Function Prototype）：

```cpp
#include <iostream>

// 函数声明（原型）——只写签名，不写函数体
int add(int a, int b);

int main() {
    std::cout << add(3, 5) << std::endl;   // 调用在定义之前，但有声明所以合法
    return 0;
}

// 函数定义
int add(int a, int b) {
    return a + b;
}
```

函数声明只需要指定返回类型、函数名和参数类型，参数名可以省略（但保留参数名有助于可读性）：

```cpp
int add(int, int);     // 合法，但不如下面的写法清晰
int add(int a, int b); // 推荐
```

在实际项目中，函数声明通常放在**头文件**（`.h` 或 `.hpp`）中，函数定义放在**源文件**（`.cpp`）中。这是 C++ 代码组织的基本模式，第十八章会详细讨论。

## 作用域

**作用域**（Scope）决定了变量在哪些位置可以被访问。

### 局部变量

在函数内部或代码块 `{}` 内声明的变量是**局部变量**（Local Variable），只在该代码块内有效：

```cpp
void demo() {
    int x = 10;   // x 的作用域从声明处开始，到函数结束
    if (x > 5) {
        int y = 20;   // y 的作用域仅限于这个 if 块
        std::cout << x + y << std::endl;
    }
    // 这里可以访问 x，但不能访问 y
}
// 这里 x 和 y 都不可访问
```

不同函数中的同名局部变量互不影响，它们是完全独立的。

### 全局变量

在所有函数之外声明的变量是**全局变量**（Global Variable），在声明之后的整个文件中都可以访问：

```cpp
int globalCount = 0;   // 全局变量

void increment() {
    ++globalCount;     // 可以访问全局变量
}

int main() {
    increment();
    increment();
    std::cout << globalCount << std::endl;   // 输出 2
    return 0;
}
```

{% hint style="warning" %}
全局变量虽然使用方便，但会导致代码难以理解和维护。任何函数都可能修改全局变量的值，使得程序状态难以追踪。应尽量避免使用全局变量，优先通过函数参数和返回值来传递数据。
{% endhint %}

### 变量遮蔽

当局部变量与外层变量同名时，局部变量会**遮蔽**（Shadow）外层变量：

```cpp
int x = 100;   // 全局变量

void demo() {
    int x = 50;   // 局部变量，遮蔽了全局的 x
    std::cout << x << std::endl;   // 输出 50，而不是 100
}
```

变量遮蔽容易引发混淆，应避免内外层使用相同的变量名。开启编译器警告 `-Wshadow` 可以帮助检测此问题。
