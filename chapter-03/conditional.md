# 3.1 条件语句

条件语句让程序根据某个条件的真假来决定执行哪段代码。C++ 提供了两种条件语句：`if` 语句和 `switch` 语句。

## if 语句

### 基本形式

```cpp
if (条件) {
    // 条件为 true 时执行
}
```

条件是一个表达式，其结果会被当作 `bool` 值来判断。如果为 `true`，执行花括号内的代码；否则跳过。

```cpp
int score = 85;

if (score >= 60) {
    std::cout << "及格" << std::endl;
}
```

### if-else

当条件不满足时需要执行另一段代码，使用 `else`：

```cpp
int score = 45;

if (score >= 60) {
    std::cout << "及格" << std::endl;
} else {
    std::cout << "不及格" << std::endl;
}
```

### if-else if-else

处理多个互斥条件时，使用 `else if` 串联：

```cpp
int score = 85;

if (score >= 90) {
    std::cout << "优秀" << std::endl;
} else if (score >= 80) {
    std::cout << "良好" << std::endl;
} else if (score >= 60) {
    std::cout << "及格" << std::endl;
} else {
    std::cout << "不及格" << std::endl;
}
```

程序从上到下依次检查每个条件，一旦某个条件为 `true`，执行对应的代码块后跳过剩余所有分支。如果所有条件都不满足，执行 `else` 中的代码（如果有的话）。

### 嵌套 if

`if` 语句可以嵌套使用，但嵌套层级过深会降低可读性：

```cpp
int age = 20;
bool hasTicket = true;

if (age >= 18) {
    if (hasTicket) {
        std::cout << "允许入场" << std::endl;
    } else {
        std::cout << "请先购票" << std::endl;
    }
} else {
    std::cout << "未满 18 岁，禁止入场" << std::endl;
}
```

上面的嵌套可以用逻辑运算符简化：

```cpp
if (age >= 18 && hasTicket) {
    std::cout << "允许入场" << std::endl;
} else if (age >= 18) {
    std::cout << "请先购票" << std::endl;
} else {
    std::cout << "未满 18 岁，禁止入场" << std::endl;
}
```

### C++17：带初始化的 if

C++17 允许在 `if` 语句的条件之前添加一个初始化语句，将变量的作用域限制在 `if-else` 块内：

```cpp
if (int remainder = number % 2; remainder == 0) {
    std::cout << number << " 是偶数" << std::endl;
} else {
    std::cout << number << " 是奇数，余数为 " << remainder << std::endl;
}
// remainder 在这里不可访问
```

这种写法避免了临时变量泄露到外层作用域。

## 条件运算符（三元运算符）

对于简单的二选一赋值，可以使用**条件运算符** `?:`，也称为**三元运算符**（Ternary Operator）：

```cpp
int score = 75;
std::string result = (score >= 60) ? "及格" : "不及格";
std::cout << result << std::endl;
```

语法为 `条件 ? 值1 : 值2`。如果条件为 `true`，表达式的值是 `值1`；否则是 `值2`。

三元运算符适合简短的条件赋值。如果逻辑较复杂，应使用 `if-else`，不要将多个三元运算符嵌套。

## switch 语句

当需要将一个变量与多个**固定常量值**逐一比较时，`switch` 比一长串 `if-else if` 更清晰：

```cpp
int day = 3;

switch (day) {
    case 1:
        std::cout << "星期一" << std::endl;
        break;
    case 2:
        std::cout << "星期二" << std::endl;
        break;
    case 3:
        std::cout << "星期三" << std::endl;
        break;
    case 4:
        std::cout << "星期四" << std::endl;
        break;
    case 5:
        std::cout << "星期五" << std::endl;
        break;
    case 6:
        std::cout << "星期六" << std::endl;
        break;
    case 7:
        std::cout << "星期日" << std::endl;
        break;
    default:
        std::cout << "无效的日期" << std::endl;
        break;
}
```

### switch 的规则

- `switch` 括号中的表达式必须是**整数类型**或**枚举类型**，不能是浮点数或字符串
- 每个 `case` 后面的值必须是**编译期常量**
- `break` 用于跳出 `switch` 块。如果省略 `break`，程序会**贯穿**（Fall Through）到下一个 `case` 继续执行
- `default` 处理所有未匹配的情况，位置不限但习惯放在最后

### 利用贯穿

有时故意省略 `break` 可以合并多个分支：

```cpp
int month = 8;

switch (month) {
    case 1: case 3: case 5: case 7:
    case 8: case 10: case 12:
        std::cout << "31 天" << std::endl;
        break;
    case 4: case 6: case 9: case 11:
        std::cout << "30 天" << std::endl;
        break;
    case 2:
        std::cout << "28 或 29 天" << std::endl;
        break;
    default:
        std::cout << "无效的月份" << std::endl;
        break;
}
```

{% hint style="warning" %}
非故意的贯穿是常见的 bug 来源。C++17 引入了 `[[fallthrough]]` 属性，用于标记故意的贯穿，帮助编译器区分是否遗漏了 `break`。
{% endhint %}

### C++17：带初始化的 switch

与 `if` 类似，C++17 也允许在 `switch` 中添加初始化语句：

```cpp
switch (int value = computeSomething(); value) {
    case 0:
        // ...
        break;
    case 1:
        // ...
        break;
    default:
        break;
}
```
