# 7.1 继承

## 为什么需要继承

假设要为一个游戏设计角色系统，有战士（Warrior）和法师（Mage）两种角色。它们有很多共同属性（名字、生命值、等级）和共同行为（移动、升级），也有各自独特的技能。

不使用继承时，代码会出现大量重复：

```cpp
class Warrior {
private:
    std::string name;
    int hp;
    int level;
    int armor;       // 战士独有

public:
    void move() { /* ... */ }
    void levelUp() { /* ... */ }
    void slash() { /* 战士技能 */ }
};

class Mage {
private:
    std::string name;   // 重复
    int hp;             // 重复
    int level;          // 重复
    int mana;           // 法师独有

public:
    void move() { /* ... 重复 */ }
    void levelUp() { /* ... 重复 */ }
    void fireball() { /* 法师技能 */ }
};
```

**继承**的思想是：将共同部分提取到一个**基类**（Base Class），让具体角色作为**派生类**（Derived Class）继承基类，只编写自己独有的部分。

## 继承的基本语法

```cpp
#include <iostream>
#include <string>

// 基类（也称父类、超类）
class Character {
protected:
    std::string name;
    int hp;
    int level;

public:
    Character(const std::string& name, int hp)
        : name(name), hp(hp), level(1) {}

    void move() {
        std::cout << name << " 正在移动" << std::endl;
    }

    void levelUp() {
        level++;
        std::cout << name << " 升到了 " << level << " 级" << std::endl;
    }

    void showStatus() const {
        std::cout << name << " [HP:" << hp << " LV:" << level << "]" << std::endl;
    }
};

// 派生类（也称子类）
class Warrior : public Character {
private:
    int armor;

public:
    Warrior(const std::string& name, int hp, int armor)
        : Character(name, hp), armor(armor) {}

    void slash() {
        std::cout << name << " 使用了劈砍！" << std::endl;
    }
};

class Mage : public Character {
private:
    int mana;

public:
    Mage(const std::string& name, int hp, int mana)
        : Character(name, hp), mana(mana) {}

    void fireball() {
        std::cout << name << " 释放了火球术！(消耗法力 " << mana << ")" << std::endl;
    }
};

int main() {
    Warrior w("亚瑟", 200, 50);
    Mage m("梅林", 120, 100);

    w.move();          // 继承自 Character
    w.showStatus();    // 继承自 Character
    w.slash();         // Warrior 独有

    m.move();          // 继承自 Character
    m.fireball();      // Mage 独有

    return 0;
}
```

派生类的声明语法为 `class 派生类 : 继承方式 基类`。`Warrior` 和 `Mage` 继承了 `Character` 的所有成员（`name`、`hp`、`level`、`move()`、`levelUp()`、`showStatus()`），同时添加了各自独有的成员。

## 三种继承方式

继承方式决定了基类成员在派生类中的**访问权限**如何变化：

| 基类成员 | public 继承 | protected 继承 | private 继承 |
|---------|------------|---------------|-------------|
| `public` | → `public` | → `protected` | → `private` |
| `protected` | → `protected` | → `protected` | → `private` |
| `private` | 不可访问 | 不可访问 | 不可访问 |

- **public 继承**：最常用，表示「是一种（is-a）」关系。基类的公开接口在派生类中仍然公开。
- **protected 继承**：基类的公开接口变为 protected，外部无法通过派生类对象访问基类接口。
- **private 继承**：基类的所有可继承成员都变为 private。表示「以...实现（implemented-in-terms-of）」关系。

{% hint style="info" %}
实际开发中，绝大多数情况使用 **public 继承**。如果你不确定该用哪种，选 public 继承。protected 和 private 继承属于进阶用法。
{% endhint %}

注意，基类的 `private` 成员无论哪种继承方式都**不可在派生类中直接访问**。这就是为什么上面的示例将 `name`、`hp`、`level` 声明为 `protected` 而非 `private`——`protected` 成员对子类可见，对外部不可见，是继承场景下常用的访问级别。

## 派生类的构造与析构

### 构造顺序

创建派生类对象时，**先构造基类部分，再构造派生类部分**。派生类的构造函数通过初始化列表调用基类的构造函数：

```cpp
class Base {
public:
    Base(int x) {
        std::cout << "Base 构造，x=" << x << std::endl;
    }
};

class Derived : public Base {
public:
    Derived(int x, int y) : Base(x) {   // 必须在初始化列表中调用基类构造函数
        std::cout << "Derived 构造，y=" << y << std::endl;
    }
};

int main() {
    Derived d(1, 2);
    // 输出：
    // Base 构造，x=1
    // Derived 构造，y=2
    return 0;
}
```

如果基类有默认构造函数，派生类可以不显式调用；否则必须在初始化列表中指明调用哪个基类构造函数。

### 析构顺序

析构顺序与构造顺序**相反**：先析构派生类部分，再析构基类部分。

```cpp
class Base {
public:
    ~Base() { std::cout << "Base 析构" << std::endl; }
};

class Derived : public Base {
public:
    ~Derived() { std::cout << "Derived 析构" << std::endl; }
};

int main() {
    Derived d;
    return 0;
}
// 输出：
// Derived 析构
// Base 析构
```

## 函数隐藏

当派生类定义了与基类**同名**的函数时（不论参数是否相同），基类的同名函数会被**隐藏**（Name Hiding）：

```cpp
class Base {
public:
    void greet() {
        std::cout << "Hello from Base" << std::endl;
    }

    void greet(const std::string& name) {
        std::cout << "Hello, " << name << " from Base" << std::endl;
    }
};

class Derived : public Base {
public:
    void greet() {    // 隐藏了 Base 中所有的 greet
        std::cout << "Hello from Derived" << std::endl;
    }
};

int main() {
    Derived d;
    d.greet();               // 调用 Derived::greet()
    // d.greet("Alice");     // 错误！Base::greet(string) 被隐藏了
    d.Base::greet("Alice");  // 正确：使用作用域解析符显式调用
    return 0;
}
```

{% hint style="warning" %}
函数隐藏与函数重载不同。重载发生在同一个作用域中，隐藏发生在基类和派生类之间。只要派生类中出现了同名函数，基类中所有同名函数（包括不同参数的重载版本）都会被隐藏。
{% endhint %}

如果希望保留基类的重载版本，可以使用 `using` 声明：

```cpp
class Derived : public Base {
public:
    using Base::greet;       // 将基类的 greet 引入派生类作用域
    void greet() {           // 同时提供自己的版本
        std::cout << "Hello from Derived" << std::endl;
    }
};
```

## 向上转型

public 继承表示「is-a」关系：每个 `Warrior` 都**是一个** `Character`。因此，基类的指针或引用可以指向派生类对象，称为**向上转型**（Upcasting）：

```cpp
void printStatus(const Character& c) {
    c.showStatus();    // 不需要知道具体类型
}

int main() {
    Warrior w("亚瑟", 200, 50);
    Mage m("梅林", 120, 100);

    printStatus(w);    // Warrior → Character&，合法
    printStatus(m);    // Mage → Character&，合法

    Character* ptr = &w;   // 基类指针指向派生类对象
    ptr->move();           // 合法
    // ptr->slash();        // 错误！基类指针只能访问基类接口

    return 0;
}
```

向上转型是安全的、隐式的。但通过基类指针/引用只能访问基类定义的成员，无法调用派生类独有的成员。这个限制将在下一节通过**虚函数**解决。

## 常见问题

**Q：继承的层数有限制吗？**

语法上没有限制，可以 A → B → C → D 多层继承。但过深的继承层次会增加理解和维护的难度，实践中一般不超过 3-4 层。

**Q：struct 可以继承吗？**

可以。`struct` 和 `class` 在继承方面的唯一区别是：`struct` 默认使用 public 继承，`class` 默认使用 private 继承。

**Q：构造函数和析构函数会被继承吗？**

不会。构造函数和析构函数不被继承，派生类必须定义自己的构造函数（可以在其中调用基类的构造函数）。
