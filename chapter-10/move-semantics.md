# 10.4 移动语义与右值引用

在前面几节学习 `std::unique_ptr` 的时候，我们看到了 `std::move` 的身影：`unique_ptr` 不能被复制，只能被“移动”。

这是围绕资源管理时，现代 C++ 的一项极其关键的特性（C++11 引入），被称为**移动语义（Move Semantics）**。

## 为什么需要移动语义？

在 C++11 之前，当赋值或传递对象时，只能进行**深拷贝（Deep Copy）**。对于包含动态分配内存或者大量数据的类（比如 `std::string` 或 `std::vector`），深拷贝非常耗费性能。

想象以下场景：

```cpp
std::vector<int> createBigData() {
    std::vector<int> data(1000000); // 100万个元素
    // ... 处理 data
    return data;
}

int main() {
    std::vector<int> myData = createBigData();
}
```

在旧标准下，`createBigData` 会在局部创建一个拥有一百万个元素的 `vector`，当它返回赋值给 `myData` 时：
1. `myData` 新开辟了一百万个元素的内存空间。
2. 进行繁重的元素拷贝工作。
3. 销毁原始临时的局部变量 `data` 的内在存储空间。

**这毫无意义！**那个临时的 `data` 马上就要被销毁了，为什么不直接把它的内存“偷”过来呢？这就是移动语义的作用：**转移资源的控制权，避免不必要的拷贝操作。**

## 左值（L-value）与右值（R-value）

理解移动语义的关键是理解什么是左值和右值。

- **左值（L-value）**：拥有持久的内存地址，可以获取地址（使用 `&`），能在多行代码中存在（变量）。
- **右值（R-value）**：没有名字的临时值，只在当前行语句存在（比如字面量、函数返回的临时对象等）。不能对其取地址。

```cpp
int a = 5; // a 是左值 (可以 &a)；5 是右值 (不能 &5)
int b = a + 10; // b 是左值；(a + 10) 是临时的右值表达式
std::string str1 = "Hello";
std::string str2 = str1 + " World"; // str1+" World" 是右值
```

## 右值引用（R-value Reference）

C++98 里面有我们熟知的引用（左值引用），使用 `&`。
C++11 引入了**右值引用**，使用 `&&` 符号，**它专门用来绑定（指向）右值/临时的对象**。

```cpp
int x = 10;
int& l_ref = x;       // 左值引用，没问题
// int& l_ref2 = 20;  // 错误：普通的左值引用不能绑定给右值

int&& r_ref = 20;     // 右值引用，专门用来绑定临时右值（20）
```

当我们有了一个右值引用，就说明我们知道：“哦，这个对象是个马上要被销毁的临时对象，我可以直接夺取（Move）它身上的资源。”

## std::move：把左值伪装成右值

有时候，像在 `std::unique_ptr` 之间移交所有权时，我们已经明确知道：“虽然 `p1` 是一个左值，但我之后再也不打算用它了，请把它当作右值处理。”

这就是 `std::move` 的作用：**它无条件地将一个对象的类型转换为右值引用类型**。它**本身不移动任何东西**，它仅仅是个类型转换操作符，目的是告诉编译器：“你可以对我使用移动语义。”

```cpp
std::string text1 = "Heavy Resource String";
// std::move 告诉编译器把 text1 当作右值
std::string text2 = std::move(text1);

// 此时 resource "Heavy Resource String" 已经转移给 text2.
// text1 被“掏空”了，它处于一个有效但未定义状态（通常为空字符串），可以被析构。
std::cout << "text1: " << text1 << "\n"; // 输出空行
std::cout << "text2: " << text2 << "\n"; // 输出 Heavy Resource String
```

## 移动构造函数与移动赋值运算符

当我们在写自己的类并管理资源时，可以通过重写类的两个特殊成员函数来主动支持“移动语义”（它们构成了与“拷贝”配对的姐妹篇）：

1. **移动构造函数（Move Constructor）**：被右值引用初始化时触发。
2. **移动赋值运算符（Move Assignment Operator）**：被右值引用赋值时触发。

规则很简单：**浅拷贝别人的指针，把别人的指针设为空。**

```cpp
class DynamicArray {
private:
    int* data;

public:
    // ...普通构造函数，析构函数，拷贝构造等忽略...

    // 移动构造函数（接收右值引用）
    DynamicArray(DynamicArray&& other) noexcept : data(other.data) {
        // "偷"走了别人的指针，然后将别人掏空
        other.data = nullptr;
        std::cout << "执行了移动构造\n";
    }

    // 移动赋值运算符
    DynamicArray& operator=(DynamicArray&& other) noexcept {
        if (this != &other) {
            delete[] data;       // 清理身上现有的资源
            data = other.data;   // 偷取资源
            other.data = nullptr;// 掏空对方
        }
        std::cout << "执行了移动赋值\n";
        return *this;
    }
};
```

{% hint style="info" %}
`noexcept` 关键字在此处极其重要：它向编译器（和标准库容器比如 `std::vector`）保证这个移动操作不会抛出异常。如果移动操作没声明为 `noexcept`，`std::vector` 扩容等场景依然会保守地退化去使用缓慢的拷贝语义（以提供强异常安全保证）。
{% endhint %}

现在，结合 `unique_ptr`，你就完全明白了什么是 `std::unique_ptr<T> p2 = std::move(p1);`。它仅仅是调用了 `unique_ptr` 类内部编写的“移动赋值运算符”，将原生指针“偷”给了 `p2`，并将 `p1` 置空罢了。