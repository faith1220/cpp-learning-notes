# 7.3 抽象类与纯虚函数

## 从设计需求说起

上一节的 `Character` 基类提供了 `attack()` 的默认实现——「进行了普通攻击」。但在实际设计中，`Character` 本身是一个抽象概念，不应该有具体的攻击方式；每种角色**必须**提供自己的攻击实现。

换句话说，我们希望：

1. `Character` 类不能被直接实例化（没有「普通角色」这种东西）
2. 所有派生类**必须**实现 `attack()` 函数，否则编译报错

**纯虚函数**和**抽象类**正是为此设计的。

## 纯虚函数

在虚函数声明末尾写上 `= 0`，该函数就成为**纯虚函数**（Pure Virtual Function）：

```cpp
class Character {
protected:
    std::string name;

public:
    Character(const std::string& name) : name(name) {}
    virtual ~Character() = default;

    virtual void attack() = 0;   // 纯虚函数：没有默认实现

    void showStatus() const {
        std::cout << name << std::endl;
    }
};
```

纯虚函数只有声明，没有函数体（通常不提供实现）。它告诉编译器：「这个函数的具体行为由派生类决定，基类不提供默认版本。」

## 抽象类

包含至少一个纯虚函数的类称为**抽象类**（Abstract Class）。抽象类**不能被实例化**：

```cpp
int main() {
    // Character c("测试");   // 编译错误！Character 是抽象类，不能创建对象
    return 0;
}
```

但抽象类可以作为指针或引用的类型，用于多态：

```cpp
Character* ptr = new Warrior("亚瑟");   // 合法
Character& ref = someWarrior;           // 合法
```

## 派生类必须实现纯虚函数

如果派生类没有实现基类的所有纯虚函数，它自身也会成为抽象类，同样不能实例化：

```cpp
class Character {
public:
    virtual void attack() = 0;
    virtual void defend() = 0;
};

class Warrior : public Character {
public:
    void attack() override {
        std::cout << "挥剑！" << std::endl;
    }
    // 没有实现 defend()
};

int main() {
    // Warrior w;   // 编译错误！Warrior 仍然是抽象类（defend 未实现）
    return 0;
}
```

只有实现了全部纯虚函数的派生类才能实例化：

```cpp
class Warrior : public Character {
public:
    void attack() override {
        std::cout << "挥剑！" << std::endl;
    }

    void defend() override {
        std::cout << "举盾防御！" << std::endl;
    }
};

Warrior w;   // 正确，所有纯虚函数都已实现
```

## 完整示例：图形类层次

```cpp
#include <iostream>
#include <cmath>
#include <vector>

// 抽象基类：图形
class Shape {
public:
    virtual ~Shape() = default;

    virtual double area() const = 0;       // 纯虚函数
    virtual double perimeter() const = 0;  // 纯虚函数
    virtual std::string typeName() const = 0;

    void print() const {
        std::cout << typeName()
                  << " 面积=" << area()
                  << " 周长=" << perimeter() << std::endl;
    }
};

// 圆形
class Circle : public Shape {
private:
    double radius;

public:
    Circle(double r) : radius(r) {}

    double area() const override {
        return 3.14159265 * radius * radius;
    }

    double perimeter() const override {
        return 2 * 3.14159265 * radius;
    }

    std::string typeName() const override {
        return "圆形";
    }
};

// 矩形
class Rectangle : public Shape {
private:
    double width, height;

public:
    Rectangle(double w, double h) : width(w), height(h) {}

    double area() const override {
        return width * height;
    }

    double perimeter() const override {
        return 2 * (width + height);
    }

    std::string typeName() const override {
        return "矩形";
    }
};

// 三角形
class Triangle : public Shape {
private:
    double a, b, c;   // 三条边

public:
    Triangle(double a, double b, double c) : a(a), b(b), c(c) {}

    double area() const override {
        double s = (a + b + c) / 2;
        return std::sqrt(s * (s - a) * (s - b) * (s - c));
    }

    double perimeter() const override {
        return a + b + c;
    }

    std::string typeName() const override {
        return "三角形";
    }
};

int main() {
    std::vector<Shape*> shapes;
    shapes.push_back(new Circle(5.0));
    shapes.push_back(new Rectangle(4.0, 6.0));
    shapes.push_back(new Triangle(3.0, 4.0, 5.0));

    for (const Shape* s : shapes) {
        s->print();    // 多态调用
    }

    // 清理内存
    for (Shape* s : shapes) {
        delete s;
    }

    return 0;
}
```

输出：

```
圆形 面积=78.5398 周长=31.4159
矩形 面积=24 周长=20
三角形 面积=6 周长=12
```

这个设计的要点：

- `Shape` 定义了统一接口（`area()`、`perimeter()`、`typeName()`）
- 每种图形实现自己的计算逻辑
- `print()` 是非虚函数，基于虚函数构建通用行为
- 新增图形类型只需继承 `Shape` 并实现纯虚函数，已有代码无需修改

## 接口类

当一个抽象类**只包含纯虚函数**（没有数据成员和非虚函数），它就起到了**接口**（Interface）的作用：

```cpp
class Serializable {
public:
    virtual ~Serializable() = default;
    virtual std::string serialize() const = 0;
    virtual void deserialize(const std::string& data) = 0;
};

class Printable {
public:
    virtual ~Printable() = default;
    virtual void print(std::ostream& out) const = 0;
};
```

C++ 没有像 Java 那样的 `interface` 关键字，但通过纯虚函数类可以达到同样的效果。一个类可以同时继承多个接口类（多重继承），这是 C++ 中多重继承最常见的正当用法，将在下一节讨论。

## 纯虚函数可以有实现

虽然不常见，但纯虚函数**可以**提供实现。派生类仍然必须重写它，但可以通过作用域解析符调用基类的版本：

```cpp
class Character {
public:
    virtual void attack() = 0;   // 仍然是纯虚函数
};

// 在类外提供实现
void Character::attack() {
    std::cout << "基础攻击动画" << std::endl;
}

class Warrior : public Character {
public:
    void attack() override {
        Character::attack();     // 调用基类提供的默认实现
        std::cout << "附加剑气效果！" << std::endl;
    }
};
```

这种技巧适用于「派生类必须重写，但可以复用一部分通用逻辑」的场景。

## 常见问题

**Q：抽象类可以有构造函数吗？**

可以。虽然抽象类不能直接实例化，但派生类构造时需要调用基类的构造函数来初始化基类部分的成员。

**Q：析构函数可以是纯虚函数吗？**

可以，但必须同时提供实现（因为派生类析构时会调用基类的析构函数）。这是一种让类成为抽象类的技巧，适用于类中没有其他适合作为纯虚函数的成员函数的情况：

```cpp
class AbstractBase {
public:
    virtual ~AbstractBase() = 0;   // 纯虚析构函数
};

AbstractBase::~AbstractBase() {}   // 必须提供实现
```

**Q：抽象类和普通基类如何选择？**

如果基类的某些函数**必须**由派生类提供具体实现，使用纯虚函数。如果基类能提供合理的默认行为，使用普通虚函数。两者可以混合使用。
