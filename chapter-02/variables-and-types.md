# 2.1 变量与数据类型

## 什么是变量

**变量**（Variable）是程序中用来存储数据的命名容器。你可以把变量想象成一个贴了标签的盒子：标签是变量名，盒子里装的东西是变量的值，而盒子的大小和形状由数据类型决定。

声明一个变量的基本语法是：

```cpp
类型 变量名;
```

例如：

```cpp
int age;         // 声明一个整数变量 age
double price;    // 声明一个双精度浮点数变量 price
char grade;      // 声明一个字符变量 grade
```

声明变量的同时可以赋予初始值，称为**初始化**（Initialization）：

```cpp
int age = 25;
double price = 9.99;
char grade = 'A';
```

C++11 引入了**统一初始化**（Uniform Initialization）语法，使用花括号：

```cpp
int age{25};
double price{9.99};
char grade{'A'};
```

花括号初始化的优势在于它会检查**窄化转换**（Narrowing Conversion）。例如 `int x{3.14};` 会产生编译错误，因为将浮点数赋给整数会丢失小数部分。而 `int x = 3.14;` 虽然也会丢失精度，但编译器只会发出警告（甚至可能不警告）。

{% hint style="warning" %}
未初始化的局部变量的值是不确定的（未定义行为）。始终在声明变量时进行初始化是一个好习惯。
{% endhint %}

## 变量命名规则

C++ 的变量名（标识符）必须遵循以下规则：

- 只能包含字母（a-z, A-Z）、数字（0-9）和下划线（`_`）
- 不能以数字开头
- 不能使用 C++ 的**关键字**（Reserved Keywords），如 `int`、`return`、`class` 等
- 区分大小写：`age`、`Age` 和 `AGE` 是三个不同的变量

合法的变量名：

```cpp
int studentAge;
int student_age;
int _count;
int x2;
```

不合法的变量名：

```cpp
int 2nd_place;    // 以数字开头
int my-name;      // 包含连字符
int class;         // 使用了关键字
```

命名建议：

- 使用有意义的名称，避免 `a`、`b`、`x` 等单字母名（循环变量除外）
- 保持风格一致：本书采用**小驼峰命名法**（camelCase），如 `studentAge`；也有项目偏好**蛇形命名法**（snake_case），如 `student_age`

## 基本数据类型

C++ 的基本数据类型分为以下几类。

### 整数类型

> **提示**：如果你对下面表格中出现的“最小宽度”、“典型大小（字节）”等底层概念感到困惑，建议跳转阅读：**[19.1 补充：理解位与字节](../chapter-19/storage-basics.md)**。

| 类型 | 最小宽度 | 典型大小 | 值范围（典型） |
|------|----------|----------|----------------|
| `short` | 16 位 | 2 字节 | -32,768 ~ 32,767 |
| `int` | 16 位 | 4 字节 | -2,147,483,648 ~ 2,147,483,647 |
| `long` | 32 位 | 4 或 8 字节 | 至少与 `int` 相同 |
| `long long` | 64 位 | 8 字节 | -9.2 x 10^18 ~ 9.2 x 10^18 |

每种整数类型都有对应的**无符号**（Unsigned）版本，只能存储非负整数，但正数范围扩大一倍：

```cpp
unsigned int count = 100;
unsigned long long bigNumber = 18446744073709551615ULL;
```

> **拓展阅读**：在 C++ 中，为了底层内存或网络协议开发，我们经常需要使用十六进制或二进制来书写整数变量，具体相关知识与转换方法请参阅：**[19.2 补充：进制的转换与书写](../chapter-19/number-bases.md)**。

{% hint style="info" %}
C++ 标准只规定了各类型的最小宽度，实际大小取决于编译器和平台。可以使用 `sizeof` 运算符查看某个类型在当前环境下占用的字节数：`sizeof(int)` 在大多数现代平台上返回 4。
{% endhint %}

### 浮点类型

用于存储带小数部分的数值：

| 类型 | 典型大小 | 有效数字位数 |
|------|----------|--------------|
| `float` | 4 字节 | 约 6-7 位 |
| `double` | 8 字节 | 约 15-16 位 |
| `long double` | 8 或 16 字节 | 至少与 `double` 相同 |

```cpp
float pi = 3.14f;          // f 后缀表示 float 字面量
double e = 2.718281828;     // 默认的小数字面量是 double
```

浮点数存在精度限制。例如 `0.1 + 0.2` 的结果并不精确等于 `0.3`，这是 IEEE 754 浮点数标准的固有特性，并非 C++ 独有的问题。因此，比较两个浮点数是否相等时应使用误差范围，而非直接用 `==`。

### 字符类型

`char` 用于存储单个字符，占 1 字节，使用单引号包裹：

```cpp
char letter = 'A';
char digit = '9';
char newline = '\n';    // 转义字符：换行
```

常见的**转义字符**（Escape Character）：

| 转义序列 | 含义 |
|----------|------|
| `\n` | 换行 |
| `\t` | 制表符（Tab） |
| `\\` | 反斜杠 |
| `\'` | 单引号 |
| `\"` | 双引号 |
| `\0` | 空字符（null） |

`char` 本质上存储的是字符的 ASCII 码值（一个整数），因此 `char` 也可以参与算术运算：

```cpp
char ch = 'A';      // ASCII 码为 65
ch = ch + 1;        // 现在 ch 是 'B'（ASCII 码 66）
```

### 布尔类型

`bool` 只有两个值：`true`（真）和 `false`（假），用于逻辑判断：

```cpp
bool isReady = true;
bool hasError = false;
```

`bool` 在内存中通常占 1 字节。在需要整数的场合，`true` 会被转换为 `1`，`false` 转换为 `0`；反过来，任何非零整数转换为 `bool` 时都是 `true`，零则为 `false`。

## 常量

如果一个值在程序运行期间不应该被修改，可以将其声明为**常量**（Constant）：

```cpp
const double PI = 3.141592653589793;
const int MAX_SIZE = 100;
```

使用 `const` 修饰的变量必须在声明时初始化，之后任何对它的赋值操作都会导致编译错误。

C++11 引入了 `constexpr`，表示该值在**编译期**（Compile Time）就能确定：

```cpp
constexpr int ARRAY_SIZE = 10;
constexpr double GRAVITY = 9.8;
```

`constexpr` 比 `const` 更严格：`const` 变量的值可以在运行期确定（例如来自用户输入），而 `constexpr` 的值必须在编译期就能计算出来。当你确定某个值是编译期常量时，优先使用 `constexpr`。

## auto 类型推导

C++11 引入了 `auto` 关键字，让编译器根据初始值自动推导变量的类型：

```cpp
auto x = 42;          // int
auto y = 3.14;        // double
auto z = 'A';         // char
auto flag = true;     // bool
```

`auto` 在类型名较长时特别有用（后续章节会遇到这种情况）。使用 `auto` 时必须提供初始值，否则编译器无法推导类型。
