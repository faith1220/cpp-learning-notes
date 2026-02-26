# 9.3 动态内存分配

## 栈与堆

程序运行时，内存被划分为几个区域，其中最重要的两个是**栈**（Stack）和**堆**（Heap）：

| 特性 | 栈 | 堆 |
|------|-----|-----|
| 分配方式 | 自动分配和释放 | 手动分配和释放 |
| 生命周期 | 随作用域自动销毁 | 直到手动释放或程序结束 |
| 大小 | 较小（通常 1-8 MB） | 较大（受系统内存限制） |
| 速度 | 极快（移动栈指针） | 较慢（需要查找空闲块） |
| 碎片 | 无 | 可能产生碎片 |

到目前为止使用的局部变量都分配在栈上。当需要在运行时决定数据大小，或需要让数据的生命周期超出当前作用域时，就需要在堆上分配内存。

## new 和 delete

C++ 使用 `new` 在堆上分配内存，`delete` 释放内存：

```cpp
// 分配单个对象
int* ptr = new int;        // 分配一个 int，值未初始化
int* ptr2 = new int(42);   // 分配并初始化为 42
int* ptr3 = new int{42};   // C++11 统一初始化语法

std::cout << *ptr2 << std::endl;   // 42

// 使用完毕后释放
delete ptr;
delete ptr2;
delete ptr3;
```

### 分配数组

```cpp
int size;
std::cout << "请输入数组大小：";
std::cin >> size;

int* arr = new int[size]();    // 分配 size 个 int，() 表示零初始化

for (int i = 0; i < size; ++i) {
    arr[i] = i * 10;
}

for (int i = 0; i < size; ++i) {
    std::cout << arr[i] << " ";
}
std::cout << std::endl;

delete[] arr;    // 释放数组，注意用 delete[]
```

{% hint style="danger" %}
`new` 分配的单个对象用 `delete` 释放，`new[]` 分配的数组用 `delete[]` 释放。混用会导致未定义行为。
{% endhint %}

## 动态对象

`new` 也可以分配自定义类型的对象，会自动调用构造函数；`delete` 会自动调用析构函数：

```cpp
class Student {
public:
    std::string name;

    Student(const std::string& n) : name(n) {
        std::cout << name << " 被创建" << std::endl;
    }

    ~Student() {
        std::cout << name << " 被销毁" << std::endl;
    }
};

int main() {
    Student* s = new Student("张三");    // 调用构造函数
    std::cout << s->name << std::endl;   // 通过 -> 访问成员

    delete s;    // 调用析构函数，然后释放内存
    return 0;
}
```

通过指针访问对象成员使用 `->` 运算符，等价于 `(*ptr).member`。

## 内存泄漏

如果用 `new` 分配了内存但忘记 `delete`，这块内存就再也无法被回收，称为**内存泄漏**（Memory Leak）：

```cpp
void leak() {
    int* ptr = new int[1000];
    // 函数结束，ptr 被销毁，但它指向的内存没有被释放
    // 这块内存泄漏了
}

int main() {
    for (int i = 0; i < 100000; ++i) {
        leak();    // 每次调用泄漏 4000 字节，累计约 400 MB
    }
    return 0;
}
```

### 常见的泄漏场景

**提前返回或异常导致跳过 delete：**

```cpp
void risky() {
    int* data = new int[100];

    if (someCondition) {
        return;          // 泄漏！data 没有被释放
    }

    // ... 使用 data ...

    delete[] data;       // 只有走到这里才释放
}
```

**指针被覆盖，丢失原地址：**

```cpp
int* ptr = new int(10);
ptr = new int(20);       // 第一块内存的地址丢失，泄漏！
delete ptr;              // 只释放了第二块
```

## 悬垂指针

释放内存后，指针仍然保存着原来的地址，但该地址的内存已经无效。这样的指针称为**悬垂指针**（Dangling Pointer）：

```cpp
int* ptr = new int(42);
delete ptr;              // 内存已释放
// ptr 仍然存储原地址，但该内存已无效

// *ptr = 10;            // 未定义行为！访问已释放的内存
```

防范措施：释放后立即将指针置空。

```cpp
delete ptr;
ptr = nullptr;           // 好习惯：释放后置空
```

## 重复释放

对同一块内存执行两次 `delete` 是未定义行为，称为**双重释放**（Double Free）：

```cpp
int* ptr = new int(42);
delete ptr;
delete ptr;    // 未定义行为！可能崩溃
```

将指针置为 `nullptr` 后再 `delete` 是安全的（`delete nullptr` 是合法的空操作）。

## 为什么不应该直接使用 new/delete

手动管理内存容易出错——泄漏、悬垂指针、双重释放都是常见的 bug 来源。现代 C++ 提供了更安全的替代方案：

| 需求 | 推荐方案 | 避免 |
|------|---------|------|
| 动态数组 | `std::vector` | `new[]` / `delete[]` |
| 动态字符串 | `std::string` | `new char[]` |
| 单个动态对象 | `std::unique_ptr`（第十章） | 裸 `new` / `delete` |
| 共享所有权 | `std::shared_ptr`（第十章） | 裸 `new` / `delete` |

{% hint style="warning" %}
理解 `new` / `delete` 的工作原理是必要的，但在实际代码中应尽量避免直接使用。第十章将介绍智能指针，它们通过 RAII 机制自动管理内存，从根本上消除泄漏和悬垂指针问题。
{% endhint %}

## 常见问题

**Q：new 失败会怎样？**

默认情况下，如果系统无法分配所需的内存，`new` 会抛出 `std::bad_alloc` 异常。也可以使用 `new(std::nothrow)` 让它在失败时返回 `nullptr` 而不是抛异常。

**Q：delete 后内存立刻被回收吗？**

`delete` 将内存归还给 C++ 运行时的内存管理器，但不一定立刻归还给操作系统。运行时可能会保留这些内存以便后续的 `new` 调用复用。

**Q：malloc/free 和 new/delete 有什么区别？**

`malloc`/`free` 是 C 语言的内存管理函数，只分配/释放原始内存。`new`/`delete` 是 C++ 的运算符，除了分配/释放内存外还会调用构造函数/析构函数。在 C++ 中应使用 `new`/`delete`，不要混用。
