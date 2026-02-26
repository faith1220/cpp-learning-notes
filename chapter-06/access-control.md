# 6.3 访问控制

## 封装的动机

回顾 6.1 节的 `BankAccount` 示例，所有成员都是 `public` 的，外部代码可以随意修改余额：

```cpp
BankAccount acc;
acc.balance = -99999;   // 合法但不合理
```

这违背了「通过存取方法操作数据」的设计意图。**封装**（Encapsulation）的核心思想是：将数据隐藏在类的内部，只通过受控的接口与外界交互。C++ 通过**访问说明符**（Access Specifier）实现这一机制。

## 三种访问级别

| 访问说明符 | 类内部 | 子类 | 类外部 |
|-----------|--------|------|--------|
| `public` | 可访问 | 可访问 | 可访问 |
| `private` | 可访问 | 不可访问 | 不可访问 |
| `protected` | 可访问 | 可访问 | 不可访问 |

- **public**：公开接口，任何代码都可以访问
- **private**：内部实现，只有类自身的成员函数可以访问
- **protected**：与 `private` 类似，但子类也可以访问（第七章继承时详细讨论）

```cpp
class BankAccount {
private:
    std::string owner;
    double balance;

public:
    BankAccount(const std::string& name, double initial)
        : owner(name), balance(initial) {}

    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    void withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
        }
    }

    double getBalance() const {
        return balance;
    }

    std::string getOwner() const {
        return owner;
    }
};

int main() {
    BankAccount acc("李四", 1000.0);
    acc.deposit(500);
    // acc.balance = -99999;       // 编译错误！balance 是 private
    std::cout << acc.getBalance() << std::endl;   // 1500
    return 0;
}
```

现在 `balance` 被标记为 `private`，外部无法直接修改。所有对余额的操作必须通过 `deposit()` 和 `withdraw()`，这两个函数内部包含了合法性检查。

## 访问说明符的作用范围

访问说明符从出现位置开始，到下一个访问说明符或类定义结束为止：

```cpp
class Example {
    // 这里是 private（class 默认）
    int a;

public:
    // 从这里开始是 public
    int b;
    void foo();

private:
    // 又回到 private
    int c;
    void bar();

public:
    // 可以多次切换
    void baz();
};
```

访问说明符可以出现任意多次，顺序也不受限制。但为了可读性，建议每种级别只使用一次，并按 `public` → `protected` → `private` 的顺序排列：

```cpp
class WellOrganized {
public:
    // 公开接口放在最前面，因为这是使用者最关心的部分
    WellOrganized();
    void doSomething();
    int getValue() const;

protected:
    // 子类可能用到的成员
    void helperForSubclass();

private:
    // 内部实现细节
    int value;
    void internalHelper();
};
```

## Getter 与 Setter

将数据设为 `private` 后，通常需要提供公开的读取和修改方法，称为 **getter** 和 **setter**：

```cpp
class Temperature {
private:
    double celsius;

public:
    Temperature(double c = 0.0) : celsius(c) {}

    // getter
    double getCelsius() const {
        return celsius;
    }

    // setter —— 可以加入校验逻辑
    void setCelsius(double c) {
        if (c < -273.15) {
            std::cout << "温度不能低于绝对零度" << std::endl;
            return;
        }
        celsius = c;
    }

    // 只读的派生数据，不需要 setter
    double getFahrenheit() const {
        return celsius * 9.0 / 5.0 + 32.0;
    }
};
```

{% hint style="info" %}
并非每个 `private` 成员都需要 getter 和 setter。只暴露外部真正需要的接口。如果一个成员只在类内部使用，就不需要提供任何访问方法。过度使用 getter/setter 反而会削弱封装的意义。
{% endhint %}

注意上面的 getter 函数末尾有一个 `const` 关键字——这表示该函数**不会修改**对象的状态，称为 **const 成员函数**。

## const 成员函数

在成员函数的参数列表后加 `const`，表示这个函数承诺不修改任何成员变量：

```cpp
class Circle {
private:
    double radius;

public:
    Circle(double r) : radius(r) {}

    // const 成员函数：只读操作
    double getRadius() const {
        return radius;
    }

    double area() const {
        return 3.14159265 * radius * radius;
    }

    // 非 const 成员函数：会修改成员
    void setRadius(double r) {
        radius = r;
    }
};
```

const 成员函数的重要性在于：**const 对象只能调用 const 成员函数**。

```cpp
void printInfo(const Circle& c) {
    std::cout << c.getRadius() << std::endl;   // 正确，getRadius() 是 const
    std::cout << c.area() << std::endl;        // 正确，area() 是 const
    // c.setRadius(5.0);                       // 错误！const 对象不能调用非 const 函数
}
```

{% hint style="warning" %}
养成习惯：所有不修改对象状态的成员函数都应该标记为 `const`。这不仅是语义上的保证，更影响函数能否被 const 引用和 const 对象调用。
{% endhint %}

## 友元

有时确实需要让类外部的函数或其他类访问 `private` 成员，C++ 提供了**友元**（Friend）机制：

### 友元函数

```cpp
class Vector2D {
private:
    double x, y;

public:
    Vector2D(double x, double y) : x(x), y(y) {}

    // 声明友元函数
    friend double distance(const Vector2D& a, const Vector2D& b);
};

// 友元函数的定义（不是成员函数，没有 Vector2D::）
double distance(const Vector2D& a, const Vector2D& b) {
    double dx = a.x - b.x;   // 可以直接访问 private 成员
    double dy = a.y - b.y;
    return std::sqrt(dx * dx + dy * dy);
}
```

### 友元类

```cpp
class Engine {
private:
    int horsepower;

public:
    Engine(int hp) : horsepower(hp) {}

    friend class Car;   // Car 类可以访问 Engine 的 private 成员
};

class Car {
public:
    void showEngine(const Engine& e) {
        std::cout << "马力：" << e.horsepower << std::endl;  // 合法
    }
};
```

{% hint style="warning" %}
友元破坏了封装性，应谨慎使用。友元关系不具有传递性（A 是 B 的友元，B 是 C 的友元，不代表 A 是 C 的友元），也不具有对称性（A 是 B 的友元，不代表 B 是 A 的友元）。
{% endhint %}

## 封装的设计原则

1. **成员变量尽量设为 `private`** — 这是封装的基本要求
2. **公开接口尽量精简** — 只暴露外部真正需要的操作
3. **const 正确性** — 不修改状态的函数一律标记 `const`
4. **慎用友元** — 优先通过公开接口交互，友元是最后手段

良好的封装使得类的内部实现可以自由修改，只要公开接口保持不变，使用这个类的代码就不需要改动。这是大型项目可维护性的基石。
