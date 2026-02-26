# 9.2 引用

## 什么是引用

**引用**（Reference）是一个已存在变量的**别名**（Alias）。引用一旦绑定到某个变量，就始终代表那个变量，对引用的所有操作等同于对原变量的操作。

```cpp
int x = 42;
int& ref = x;    // ref 是 x 的引用（别名）

std::cout << ref << std::endl;   // 42
ref = 100;
std::cout << x << std::endl;    // 100（通过 ref 修改了 x）
```

声明语法：`类型& 变量名 = 初始值`。`int&` 表示「int 的引用」。

## 引用的基本规则

1. **必须初始化** — 引用在声明时必须绑定到一个对象，不能只声明不初始化

```cpp
int& ref;         // 编译错误！引用必须初始化
```

2. **不能重新绑定** — 引用一旦绑定，就不能改为引用其他对象

```cpp
int a = 10, b = 20;
int& ref = a;
ref = b;          // 这是把 b 的值赋给 a，不是让 ref 引用 b
std::cout << a << std::endl;   // 20（a 的值被修改了）
```

3. **没有空引用** — 引用必须绑定到有效对象，不存在「空引用」的概念

4. **引用不是对象** — 引用本身不占用独立的内存空间（编译器在底层通常用指针实现，但语义上引用不是对象）

## 引用作为函数参数

引用最常见的用途是作为函数参数，实现**按引用传递**（Pass by Reference）：

```cpp
// 按值传递：函数内部操作的是副本
void swapByValue(int a, int b) {
    int temp = a;
    a = b;
    b = temp;
    // 修改的是副本，对原变量无影响
}

// 按引用传递：函数内部操作的是原变量
void swapByRef(int& a, int& b) {
    int temp = a;
    a = b;
    b = temp;
    // 直接修改原变量
}

int main() {
    int x = 1, y = 2;

    swapByValue(x, y);
    std::cout << x << " " << y << std::endl;   // 1 2（未交换）

    swapByRef(x, y);
    std::cout << x << " " << y << std::endl;   // 2 1（已交换）

    return 0;
}
```

### const 引用

如果函数不需要修改参数，应使用 **const 引用**，既避免拷贝又防止意外修改：

```cpp
void print(const std::string& s) {    // 不拷贝，不修改
    std::cout << s << std::endl;
    // s = "test";    // 编译错误！const 引用不能修改
}
```

const 引用还有一个特殊能力：可以绑定到**右值**（临时对象）：

```cpp
const int& ref = 42;              // 合法：const 引用可以绑定字面量
const std::string& r = "hello";   // 合法：绑定临时 string 对象

// int& ref2 = 42;                // 错误！非 const 引用不能绑定右值
```

## 引用作为返回值

函数可以返回引用，避免返回值的拷贝：

```cpp
class IntArray {
private:
    int data[100];

public:
    int& at(int index) {
        return data[index];    // 返回引用，调用者可以直接修改
    }

    const int& at(int index) const {
        return data[index];    // const 版本，只读
    }
};

int main() {
    IntArray arr;
    arr.at(0) = 42;     // 通过返回的引用赋值
    std::cout << arr.at(0) << std::endl;   // 42
    return 0;
}
```

{% hint style="danger" %}
与指针一样，**不要返回局部变量的引用**。局部变量在函数返回后被销毁，引用变成悬垂引用。
{% endhint %}

## 引用与指针的对比

| 特性 | 引用 | 指针 |
|------|------|------|
| 初始化 | 必须初始化 | 可以不初始化（但不推荐） |
| 空值 | 不存在空引用 | 可以为 `nullptr` |
| 重新绑定 | 不可以 | 可以指向其他对象 |
| 语法 | 直接使用，无需 `*` 或 `&` | 需要 `*` 解引用、`&` 取地址 |
| 算术运算 | 不支持 | 支持（指针加减） |
| 内存占用 | 通常不独立占用 | 占用一个指针大小 |
| 多级间接 | 没有「引用的引用」 | 有「指针的指针」（`int**`） |

### 选择指南

- **优先使用引用** — 大多数参数传递和返回值场景
- **使用指针的场景**：
  - 需要表示「可能为空」的语义
  - 需要在运行时切换指向的对象
  - 需要指针算术（遍历数组等）
  - 与 C 语言接口交互
  - 动态内存管理（`new` / `delete`）

{% hint style="info" %}
现代 C++ 的建议：函数参数优先用 const 引用传递大对象，用值传递小对象和基本类型。需要修改参数时用非 const 引用。只在确实需要指针语义时才使用指针。
{% endhint %}
