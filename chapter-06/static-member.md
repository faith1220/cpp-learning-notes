# 6.5 静态成员

## 从问题出发

假设需要统计程序中创建了多少个 `Player` 对象。用普通成员变量无法实现，因为每个对象都有自己独立的副本：

```cpp
class Player {
public:
    int count;   // 每个对象都有一份，无法用于全局计数
};
```

需要一种「属于类本身、而非某个对象」的成员。这就是**静态成员**（Static Member）。

## 静态成员变量

使用 `static` 关键字声明的成员变量，被该类的所有对象**共享**：

```cpp
#include <iostream>
#include <string>

class Player {
private:
    std::string name;

public:
    static int count;   // 声明静态成员变量

    Player(const std::string& n) : name(n) {
        count++;
        std::cout << name << " 加入，当前玩家数：" << count << std::endl;
    }

    ~Player() {
        count--;
        std::cout << name << " 离开，当前玩家数：" << count << std::endl;
    }
};

// 在类外定义并初始化静态成员变量（必须）
int Player::count = 0;

int main() {
    Player a("Alice");     // Alice 加入，当前玩家数：1
    Player b("Bob");       // Bob 加入，当前玩家数：2
    {
        Player c("Carol"); // Carol 加入，当前玩家数：3
    }                      // Carol 离开，当前玩家数：2
    std::cout << "最终玩家数：" << Player::count << std::endl;  // 2
    return 0;
}
```

### 关键要点

1. **类内声明，类外定义** — 静态成员变量在类内用 `static` 声明，但必须在类外提供定义和初始化。这是因为静态成员不属于任何对象，需要在全局作用域中分配存储空间。

2. **所有对象共享同一份** — 无论创建多少个 `Player` 对象，`count` 只有一份。

3. **两种访问方式**：
   - 通过类名：`Player::count`（推荐）
   - 通过对象：`a.count`（合法但不推荐，容易误导读者以为是普通成员）

### 类内初始化的特例

C++17 允许使用 `inline` 关键字在类内直接初始化静态成员变量：

```cpp
class Config {
public:
    // C++17：inline 静态成员可以类内初始化
    inline static int maxRetries = 3;
    inline static std::string version = "1.0.0";
};
```

对于 `const` 整型或枚举类型的静态成员，C++11 之前就允许类内初始化：

```cpp
class Limits {
public:
    static const int maxSize = 100;        // 整型 const 可以类内初始化
    static constexpr double pi = 3.14159;  // constexpr 也可以
};
```

## 静态成员函数

**静态成员函数**也属于类而非对象，使用 `static` 关键字声明。它与普通成员函数最大的区别是：**没有 `this` 指针**。

```cpp
class MathUtil {
public:
    static double square(double x) {
        return x * x;
    }

    static double cube(double x) {
        return x * x * x;
    }
};

int main() {
    // 通过类名调用，不需要创建对象
    std::cout << MathUtil::square(5) << std::endl;   // 25
    std::cout << MathUtil::cube(3) << std::endl;     // 27
    return 0;
}
```

由于没有 `this` 指针，静态成员函数有以下限制：

- **不能访问非静态成员变量** — 非静态成员属于具体对象，没有 `this` 就不知道访问哪个对象的
- **不能调用非静态成员函数** — 同理
- **不能声明为 `const`** — `const` 修饰的是 `this` 指向的对象，没有 `this` 也就无从 `const`

```cpp
class Example {
private:
    int value;              // 非静态成员
    static int shared;      // 静态成员

public:
    static void staticFunc() {
        shared = 10;        // 正确：可以访问静态成员
        // value = 10;      // 错误：不能访问非静态成员
    }

    void normalFunc() {
        value = 10;         // 正确：可以访问非静态成员
        shared = 10;        // 正确：也可以访问静态成员
    }
};

int Example::shared = 0;
```

## 静态成员的典型应用

### 对象计数

```cpp
class Connection {
private:
    static int activeCount;

public:
    Connection() { activeCount++; }
    ~Connection() { activeCount--; }

    static int getActiveCount() {
        return activeCount;
    }
};

int Connection::activeCount = 0;
```

### 工厂方法

静态成员函数常用于实现**工厂方法**（Factory Method），提供比构造函数更灵活的对象创建方式：

```cpp
class Color {
private:
    int r, g, b;

    Color(int r, int g, int b) : r(r), g(g), b(b) {}

public:
    // 工厂方法：语义更清晰的创建方式
    static Color red()   { return Color(255, 0, 0); }
    static Color green() { return Color(0, 255, 0); }
    static Color blue()  { return Color(0, 0, 255); }

    static Color fromHex(int hex) {
        return Color((hex >> 16) & 0xFF,
                     (hex >> 8) & 0xFF,
                     hex & 0xFF);
    }

    void print() const {
        std::cout << "(" << r << ", " << g << ", " << b << ")" << std::endl;
    }
};

int main() {
    Color c1 = Color::red();
    Color c2 = Color::fromHex(0x1A2B3C);
    c1.print();   // (255, 0, 0)
    c2.print();   // (26, 43, 60)
    return 0;
}
```

### 全局配置 / 单例模式的雏形

```cpp
class AppConfig {
private:
    inline static std::string appName = "MyApp";
    inline static int logLevel = 1;

public:
    static void setAppName(const std::string& name) { appName = name; }
    static std::string getAppName() { return appName; }

    static void setLogLevel(int level) { logLevel = level; }
    static int getLogLevel() { return logLevel; }
};

int main() {
    AppConfig::setAppName("学习助手");
    std::cout << AppConfig::getAppName() << std::endl;
    return 0;
}
```

## 静态成员与非静态成员对比

| 特性 | 非静态成员 | 静态成员 |
|------|-----------|---------|
| 归属 | 属于对象 | 属于类 |
| 存储 | 每个对象各一份 | 全局仅一份 |
| 访问方式 | `obj.member` | `ClassName::member`（推荐） |
| this 指针 | 有 | 无 |
| 访问其他成员 | 可访问静态和非静态 | 只能访问静态 |
| const 修饰 | 支持 | 不支持（函数） |

## 常见问题

**Q：静态成员变量的内存何时分配？**

静态成员变量在程序启动时分配（位于全局/静态存储区），在 `main()` 执行前初始化，程序结束时销毁。它的生命周期与程序相同，不随对象的创建和销毁而变化。

**Q：静态成员变量可以是 private 的吗？**

可以。访问控制（public/private/protected）对静态成员同样有效。private 的静态成员只能通过类的成员函数（包括静态成员函数）访问。

**Q：头文件中如何处理静态成员变量的定义？**

传统做法是在头文件中声明，在一个 `.cpp` 文件中定义，避免重复定义错误。C++17 的 `inline static` 允许直接在头文件中定义，编译器保证只有一份实例。
