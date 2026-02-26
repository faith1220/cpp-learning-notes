# 7.2 虚函数与多态

## 问题引入

上一节提到，基类指针可以指向派生类对象（向上转型）。但如果基类和派生类有同名函数，通过基类指针调用时会发生什么？

```cpp
#include <iostream>
#include <string>

class Character {
protected:
    std::string name;

public:
    Character(const std::string& name) : name(name) {}

    void attack() {
        std::cout << name << " 进行了普通攻击" << std::endl;
    }
};

class Warrior : public Character {
public:
    Warrior(const std::string& name) : Character(name) {}

    void attack() {
        std::cout << name << " 挥剑猛砍！" << std::endl;
    }
};

class Mage : public Character {
public:
    Mage(const std::string& name) : Character(name) {}

    void attack() {
        std::cout << name << " 释放魔法！" << std::endl;
    }
};

int main() {
    Warrior w("亚瑟");
    Mage m("梅林");

    Character* ptr1 = &w;
    Character* ptr2 = &m;

    ptr1->attack();   // 输出：亚瑟 进行了普通攻击
    ptr2->attack();   // 输出：梅林 进行了普通攻击

    return 0;
}
```

尽管 `ptr1` 实际指向 `Warrior` 对象，调用的却是 `Character::attack()`。这是因为编译器在编译期根据指针的**声明类型**（`Character*`）决定调用哪个函数——这称为**静态绑定**（Static Binding）或**早绑定**（Early Binding）。

我们期望的行为是：根据指针指向的**实际对象类型**来选择函数。这需要**虚函数**。

## 虚函数

在基类的函数声明前加上 `virtual` 关键字，该函数就成为**虚函数**（Virtual Function）：

```cpp
class Character {
protected:
    std::string name;

public:
    Character(const std::string& name) : name(name) {}

    virtual void attack() {    // 加上 virtual
        std::cout << name << " 进行了普通攻击" << std::endl;
    }
};

class Warrior : public Character {
public:
    Warrior(const std::string& name) : Character(name) {}

    void attack() {            // 自动成为虚函数（即使不写 virtual）
        std::cout << name << " 挥剑猛砍！" << std::endl;
    }
};

class Mage : public Character {
public:
    Mage(const std::string& name) : Character(name) {}

    void attack() {
        std::cout << name << " 释放魔法！" << std::endl;
    }
};

int main() {
    Warrior w("亚瑟");
    Mage m("梅林");

    Character* ptr1 = &w;
    Character* ptr2 = &m;

    ptr1->attack();   // 输出：亚瑟 挥剑猛砍！
    ptr2->attack();   // 输出：梅林 释放魔法！

    return 0;
}
```

加上 `virtual` 后，程序在运行时根据指针实际指向的对象类型来决定调用哪个版本的 `attack()`。这称为**动态绑定**（Dynamic Binding）或**晚绑定**（Late Binding），也就是**多态**（Polymorphism）。

## 多态的威力

多态的核心价值在于：编写只依赖基类接口的通用代码，无需关心具体类型。

```cpp
#include <iostream>
#include <string>
#include <vector>

void battleRound(std::vector<Character*>& team) {
    for (Character* c : team) {
        c->attack();    // 每个角色调用自己的 attack 版本
    }
}

int main() {
    Warrior w1("亚瑟");
    Warrior w2("兰斯洛特");
    Mage m1("梅林");

    std::vector<Character*> team = {&w1, &m1, &w2};
    battleRound(team);
    // 输出：
    // 亚瑟 挥剑猛砍！
    // 梅林 释放魔法！
    // 兰斯洛特 挥剑猛砍！

    return 0;
}
```

`battleRound()` 函数不需要知道队伍中有哪些具体角色类型，只要它们都是 `Character` 的派生类并实现了 `attack()`。未来新增角色类型（如弓箭手）时，`battleRound()` 完全不需要修改。

## override 关键字（C++11）

派生类重写虚函数时，推荐在函数声明末尾加上 `override`：

```cpp
class Warrior : public Character {
public:
    Warrior(const std::string& name) : Character(name) {}

    void attack() override {    // 明确标记为重写
        std::cout << name << " 挥剑猛砍！" << std::endl;
    }
};
```

`override` 的作用是让编译器**检查**该函数是否确实重写了基类的虚函数。如果函数签名不匹配（例如参数类型写错、拼写错误），编译器会报错，而不是静默地创建一个新函数：

```cpp
class Warrior : public Character {
public:
    // 假设基类是 virtual void attack()
    void atack() override {     // 编译错误！基类没有 atack() 虚函数
        // ...                  // 拼写错误被立刻发现
    }

    void attack(int damage) override {  // 编译错误！参数不匹配
        // ...
    }
};
```

{% hint style="warning" %}
始终对重写的虚函数使用 `override`。这是现代 C++ 的基本实践，可以防止因拼写错误或签名不匹配导致的隐蔽 bug。
{% endhint %}

## final 关键字（C++11）

`final` 有两种用法：

**禁止重写某个虚函数：**

```cpp
class Character {
public:
    virtual void attack() {}
};

class Warrior : public Character {
public:
    void attack() override final {   // Warrior 的子类不能再重写 attack
        std::cout << "挥剑！" << std::endl;
    }
};

class EliteWarrior : public Warrior {
public:
    // void attack() override {}    // 编译错误！attack 已被标记为 final
};
```

**禁止继承某个类：**

```cpp
class Singleton final {    // 不允许任何类继承 Singleton
    // ...
};

// class MySingleton : public Singleton {};   // 编译错误！
```

## 虚析构函数

当通过基类指针 `delete` 派生类对象时，如果基类的析构函数不是虚函数，只会调用基类的析构函数，派生类的析构函数不会被调用，可能导致资源泄漏：

```cpp
class Base {
public:
    ~Base() {   // 非虚析构函数
        std::cout << "Base 析构" << std::endl;
    }
};

class Derived : public Base {
private:
    int* data;

public:
    Derived() : data(new int[100]) {}

    ~Derived() {
        delete[] data;
        std::cout << "Derived 析构" << std::endl;
    }
};

int main() {
    Base* ptr = new Derived();
    delete ptr;    // 只调用 Base::~Base()，Derived 的 data 内存泄漏！
    return 0;
}
```

解决方法是将基类的析构函数声明为 `virtual`：

```cpp
class Base {
public:
    virtual ~Base() {
        std::cout << "Base 析构" << std::endl;
    }
};
```

现在 `delete ptr` 会先调用 `Derived::~Derived()`，再调用 `Base::~Base()`，正确释放所有资源。

{% hint style="danger" %}
**规则：只要一个类打算作为基类被继承，就应该将其析构函数声明为 `virtual`。** 这是 C++ 最重要的实践准则之一。
{% endhint %}

## 虚函数的工作原理（简述）

编译器通过**虚函数表**（Virtual Table，简称 vtable）实现动态绑定：

1. 每个包含虚函数的类都有一张虚函数表，表中存储该类各虚函数的地址
2. 每个对象内部有一个隐藏的指针（vptr），指向所属类的虚函数表
3. 通过基类指针调用虚函数时，程序先通过 vptr 找到虚函数表，再从表中取出正确的函数地址

```
Character 的 vtable          Warrior 的 vtable
+-------------------+       +-------------------+
| attack → Character|       | attack → Warrior  |
|         ::attack()|       |         ::attack() |
+-------------------+       +-------------------+

Character* ptr = &warrior;
ptr->attack();
  → ptr->vptr → Warrior 的 vtable → Warrior::attack()
```

这意味着虚函数的调用比普通函数调用多一次指针间接寻址，有微小的性能开销。同时，每个对象会多占用一个指针大小的内存（通常 8 字节）。对于绝大多数应用，这个开销可以忽略。

## 常见问题

**Q：构造函数可以是虚函数吗？**

不可以。对象在构造过程中 vptr 尚未完全设置好，无法进行动态分发。如果需要类似「虚构造」的效果，可以使用工厂方法模式。

**Q：在构造函数和析构函数中调用虚函数会怎样？**

不会发生多态。在基类构造函数执行期间，对象的类型被视为基类（vptr 指向基类的 vtable），因此虚函数调用会解析到基类版本。析构函数中同理。这是一个常见的陷阱。

**Q：所有函数都应该声明为 virtual 吗？**

不建议。虚函数有性能开销（虽然很小），更重要的是语义上的考量——只有那些**预期会被派生类重写**的函数才应声明为虚函数。不需要重写的函数保持为非虚函数，既清晰又高效。
