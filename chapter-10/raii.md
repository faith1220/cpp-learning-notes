# 10.1 RAII 原则

在现代 C++ 中，**RAII（Resource Acquisition Is Initialization，资源获取即初始化）** 是最重要的编程范式之一。如果你只记住 C++ 中的一个核心概念，那就应该是 RAII。

## 什么是 RAII？

RAII 这个名字由于历史原因起得有点反直觉，其核心思想可以概括为：**将资源（内存、文件句柄、网络连接、互斥锁等）的生命周期与对象的生命周期绑定。**

具体做法是：
1. **在类的构造函数（Constructor）中获取（Acquire）资源。**
2. **在类的析构函数（Destructor）中释放（Release）资源。**

由于 C++ 保证了在离开作用域时，栈上局部变量的析构函数一定会被自动调用，这就保证了资源一定会被释放，即使中间发生了异常或提前 `return`。

## 为什么需要 RAII？

回忆一下我们在使用原生指针和原生资源时面临的问题：

```cpp
void processFile(const std::string& filename) {
    FILE* file = fopen(filename.c_str(), "r"); // 获取资源
    if (!file) return;

    // ... 一些操作 ...

    if (error_occurred) {
        fclose(file); // 早期返回时容易忘记释放，导致资源泄漏
        return;
    }

    fclose(file); // 正常结束时释放资源
}
```

在上面的代码中，开发者必须在多处手动调用 `fclose()`。如果以后有人在这里加了抛出异常的代码，或者加了新的 `return`，很容易就会漏掉释放资源的操作。

## 使用 RAII 改造代码

让我们用一个简单的 RAII 类来包装文件指针：

```cpp
#include <iostream>

class FileHandler {
private:
    FILE* file;

public:
    // 构造函数：获取资源
    FileHandler(const std::string& filename, const char* mode) {
        file = fopen(filename.c_str(), mode);
        std::cout << "文件已打开\n";
    }

    // 析构函数：释放资源
    ~FileHandler() {
        if (file) {
            fclose(file);
            std::cout << "文件已被自动关闭\n";
        }
    }

    FILE* get() const { return file; }
    bool isOpen() const { return file != nullptr; }
};

void processFile(const std::string& filename) {
    FileHandler fh(filename, "r"); // 离开作用域时自动关闭

    if (!fh.isOpen()) return;

    if (true) { // 模拟发生错误或早期返回
        std::cout << "发生错误，提前退出\n";
        return;     // 此时 fh 的析构函数会被调用！
    }
}
```

{% hint style="success" %}
在上述代码中，无论 `processFile` 函数是如何退出的（正常返回、提前返回、抛出异常），`fh`对象的析构函数都会被调用，文件一定会被安全关闭。这就是 RAII 的魔力。
{% endhint %}

## 标准库中的 RAII

在 C++ 标准库中，几乎所有的容器和现代实用类都遵循 RAII 原则：

- **`std::string` 和 `std::vector`**：内部分配堆内存，离开作用域时在析构函数中自动 `delete[]` 释放。
- **`std::fstream`**：文件流，构造时打开文件，析构时关闭文件。
- **`std::lock_guard`**：用于多线程，构造时加锁，析构时解锁。
- **智能指针（Smart Pointers）**：将在下面几节介绍，负责管理动态内存的 RAII 包装器。

## 常见问题

**Q: 只有内存才需要使用 RAII 吗？**
A: 虽然我们在谈论 RAII 时经常谈到内存泄漏，但 RAII 适用于**任何**有限的资源，包括：
- 文件描述符/句柄
- 数据库连接
- 网络 Socket
- 动态分配的内存
- 多线程中的互斥锁（Mutex）