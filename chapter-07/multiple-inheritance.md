# 7.4 多重继承

## 什么是多重继承

前面的示例中，每个派生类只有一个基类，这称为**单继承**（Single Inheritance）。C++ 还允许一个类同时继承**多个**基类，称为**多重继承**（Multiple Inheritance）：

```cpp
class A { /* ... */ };
class B { /* ... */ };

class C : public A, public B {   // C 同时继承 A 和 B
    // ...
};
```

## 基本用法

```cpp
#include <iostream>
#include <string>

class Flyable {
public:
    void fly() {
        std::cout << "在天空飞翔" << std::endl;
    }
};

class Swimmable {
public:
    void swim() {
        std::cout << "在水中游泳" << std::endl;
    }
};

class Duck : public Flyable, public Swimmable {
public:
    void quack() {
        std::cout << "嘎嘎叫" << std::endl;
    }
};

int main() {
    Duck d;
    d.fly();     // 来自 Flyable
    d.swim();    // 来自 Swimmable
    d.quack();   // Duck 自己的
    return 0;
}
```

`Duck` 同时继承了 `Flyable` 和 `Swimmable`，拥有两个基类的所有成员。

## 构造与析构顺序

多重继承时，基类的构造顺序由**继承列表中的声明顺序**决定（从左到右），与初始化列表中的书写顺序无关。析构顺序与构造顺序相反。

```cpp
class A {
public:
    A() { std::cout << "A 构造" << std::endl; }
    ~A() { std::cout << "A 析构" << std::endl; }
};

class B {
public:
    B() { std::cout << "B 构造" << std::endl; }
    ~B() { std::cout << "B 析构" << std::endl; }
};

class C : public A, public B {   // A 先于 B
public:
    C() { std::cout << "C 构造" << std::endl; }
    ~C() { std::cout << "C 析构" << std::endl; }
};

int main() {
    C obj;
    return 0;
}
// 输出：
// A 构造
// B 构造
// C 构造
// C 析构
// B 析构
// A 析构
```

## 名称冲突与二义性

当多个基类有同名成员时，直接访问会产生**二义性**（Ambiguity）错误：

```cpp
class Printer {
public:
    void start() {
        std::cout << "打印机启动" << std::endl;
    }
};

class Scanner {
public:
    void start() {
        std::cout << "扫描仪启动" << std::endl;
    }
};

class MultiFunctionDevice : public Printer, public Scanner {
    // 继承了两个 start()
};

int main() {
    MultiFunctionDevice mfd;
    // mfd.start();              // 编译错误！二义性，不知道调用哪个 start
    mfd.Printer::start();        // 正确：明确指定
    mfd.Scanner::start();        // 正确：明确指定
    return 0;
}
```

解决方法是使用**作用域解析符**指定调用哪个基类的版本，或者在派生类中重新定义该函数：

```cpp
class MultiFunctionDevice : public Printer, public Scanner {
public:
    void start() {
        Printer::start();
        Scanner::start();
        std::cout << "多功能设备就绪" << std::endl;
    }
};
```

## 菱形继承问题

多重继承最著名的问题是**菱形继承**（Diamond Inheritance），也称**钻石问题**（Diamond Problem）：

```
      Animal
      /    \
   Flyable  Swimmable
      \    /
       Duck
```

```cpp
class Animal {
public:
    std::string name;
    Animal(const std::string& n) : name(n) {}
};

class Flyable : public Animal {
public:
    Flyable(const std::string& n) : Animal(n) {}
    void fly() { std::cout << name << " 在飞" << std::endl; }
};

class Swimmable : public Animal {
public:
    Swimmable(const std::string& n) : Animal(n) {}
    void swim() { std::cout << name << " 在游" << std::endl; }
};

class Duck : public Flyable, public Swimmable {
public:
    Duck(const std::string& n) : Flyable(n), Swimmable(n) {}
};
```

问题在于：`Duck` 对象中有**两份** `Animal` 的数据（一份来自 `Flyable`，一份来自 `Swimmable`）。这会导致：

1. **内存浪费** — `name` 被存储了两次
2. **二义性** — `duck.name` 不知道访问哪一份
3. **数据不一致** — 修改一份不会影响另一份

```cpp
Duck d("唐老鸭");
// d.name;                  // 编译错误！二义性
d.Flyable::name;            // 一份 name
d.Swimmable::name;          // 另一份 name（两份独立存在）
```

## 虚继承

C++ 通过**虚继承**（Virtual Inheritance）解决菱形继承问题。在中间层的继承声明中加上 `virtual` 关键字：

```cpp
class Animal {
public:
    std::string name;
    Animal(const std::string& n) : name(n) {
        std::cout << "Animal 构造：" << name << std::endl;
    }
};

class Flyable : virtual public Animal {      // 虚继承
public:
    Flyable(const std::string& n) : Animal(n) {}
    void fly() { std::cout << name << " 在飞" << std::endl; }
};

class Swimmable : virtual public Animal {    // 虚继承
public:
    Swimmable(const std::string& n) : Animal(n) {}
    void swim() { std::cout << name << " 在游" << std::endl; }
};

class Duck : public Flyable, public Swimmable {
public:
    // 虚继承时，最终派生类必须直接调用虚基类的构造函数
    Duck(const std::string& n)
        : Animal(n), Flyable(n), Swimmable(n) {}
};

int main() {
    Duck d("唐老鸭");
    d.name = "唐老鸭";        // 不再有二义性，只有一份 Animal
    d.fly();                  // 唐老鸭 在飞
    d.swim();                 // 唐老鸭 在游
    return 0;
}
```

虚继承确保 `Duck` 中只有**一份** `Animal` 子对象。

### 虚继承的关键规则

1. **虚基类由最终派生类构造** — `Duck` 必须在初始化列表中直接调用 `Animal` 的构造函数，即使 `Animal` 不是它的直接基类。`Flyable` 和 `Swimmable` 中对 `Animal` 构造函数的调用会被忽略。

2. **构造顺序** — 虚基类总是最先构造，然后按照继承列表的顺序构造非虚基类，最后构造派生类自身。

3. **性能开销** — 虚继承引入了额外的间接寻址（通过虚基类指针/偏移表），访问虚基类成员略慢于普通继承。

## 多重继承的实践建议

多重继承功能强大但容易引发复杂问题。以下是实践中的推荐做法：

### 推荐用法：继承多个接口类

```cpp
class Serializable {
public:
    virtual ~Serializable() = default;
    virtual std::string serialize() const = 0;
};

class Drawable {
public:
    virtual ~Drawable() = default;
    virtual void draw() const = 0;
};

class Widget : public Serializable, public Drawable {
private:
    std::string label;

public:
    Widget(const std::string& l) : label(l) {}

    std::string serialize() const override {
        return "Widget:" + label;
    }

    void draw() const override {
        std::cout << "[" << label << "]" << std::endl;
    }
};
```

接口类（只含纯虚函数）没有数据成员，不会产生菱形继承的数据重复问题。这是多重继承最安全、最常见的用法。

### 应避免的用法

- 多个包含数据成员的基类
- 深层次的菱形继承
- 过度依赖虚继承来修补设计问题

## 多重继承 vs 组合

很多时候，多重继承可以用**组合**（Composition）替代，而且通常更清晰：

```cpp
// 多重继承方案
class FlyingSwimmingAnimal : public Flyable, public Swimmable {
    // ...
};

// 组合方案（通常更好）
class Duck {
private:
    FlyAbility flyAbility;     // 有一个飞行能力
    SwimAbility swimAbility;   // 有一个游泳能力

public:
    void fly() { flyAbility.execute(); }
    void swim() { swimAbility.execute(); }
};
```

{% hint style="info" %}
设计准则：优先使用组合而非继承（Prefer composition over inheritance）。只在「is-a」关系明确时使用继承，「has-a」或「can-do」关系用组合。多重继承主要用于实现多个接口。
{% endhint %}

## 常见问题

**Q：Java 和 C# 为什么不支持多重继承？**

正是因为菱形继承等问题的复杂性。Java 和 C# 选择了只允许单继承 + 多接口实现的方案，用 `interface` 关键字定义纯接口。C++ 的多重继承更灵活，但也需要程序员自己处理复杂性。

**Q：虚继承会影响性能吗？**

会有一定影响。虚继承需要额外的指针或偏移表来定位虚基类子对象，访问虚基类成员时多了一次间接寻址。对于性能敏感的代码需要考虑这个开销，但大多数场景可以忽略。

**Q：什么时候应该使用多重继承？**

最常见的正当用法是让一个类实现多个接口（纯抽象基类）。对于非接口的多重继承，建议先考虑能否用组合替代。如果确实需要，注意避免菱形继承或正确使用虚继承。
