# 16.1 异常机制

C++ 异常处理主要依赖三个关键字：`throw`（抛出）、`try`（尝试执行块）和 `catch`（捕获块）。

## 基础语法

当你的函数在执行过程中遇到了无法内部自行决定如何解决的烂摊子时，它可以用 `throw` 把任何类型的数据“扔”出去，同时立刻终止自身的运行。

```cpp
#include <iostream>
#include <string>

// 一个做除法的函数
double divide(double a, double b) {
    if (b == 0.0) {
        // 发现错误，立刻抛出一个字符串异常！
        throw std::string("除数不能为 0！");
    }
    return a / b;
}

int main() {
    // try 块包裹可能发生风险的代码
    try {
        std::cout << "5 / 2 = " << divide(5.0, 2.0) << "\n";

        // 下面这行会触发异常，且后面的打印代码永远不会被执行
        std::cout << "5 / 0 = " << divide(5.0, 0.0) << "\n";
        std::cout << "这行字不会出现在屏幕上。\n";

    }
    // catch 块负责接住特定类型的异常对象
    catch (const std::string& error_msg) {
        std::cerr << "出错了：捕获到异常 -> " << error_msg << "\n";
        // 在这里进行抢救或者恢复操作...
    }

    std::cout << "程序抢救成功，继续执行其它任务...\n";
    return 0;
}
```

## 栈展开（Stack Unwinding）机制

这是 C++ 异常最核心、也区别于别的语言的最重要特征。

当一个异常被 `throw` 抛出后，程序运行会立刻暂停当前函数，转而开始沿着函数调用栈**向外寻找**匹配的 `catch` 块。
在这个原路退回的“寻找过程”中，**所有遇到了的、处在目前还未退出的作用域内的局部对象，它们的析构函数都会被依次且正确地调用！**

这个“沿途退回并自动清理对象”的机制，就被称为 **栈展开（Stack Unwinding）**。

```cpp
#include <iostream>

class TestObj {
    int id;
public:
    TestObj(int i) : id(i) { std::cout << "构造对象 " << id << "\n"; }
    ~TestObj() { std::cout << "析构对象 " << id << "\n"; }
};

void dangerFunction() {
    TestObj obj2(2); // 局部对象
    std::cout << "准备抛出异常！\n";
    throw 404;       // 抛出一个 int 型的异常错误码
}

void middleFunction() {
    TestObj obj1(1); // 局部对象
    dangerFunction();
}

int main() {
    try {
        middleFunction();
    } catch (int errCode) {
        std::cout << "接到了错误码: " << errCode << "\n";
    }
}

// 打印运行结果将会是：
// 构造对象 1
// 构造对象 2
// 准备抛出异常！
// 析构对象 2      <-- 栈展开的功劳！
// 析构对象 1      <-- 栈展开的功劳！
// 接到了错误码: 404
```

正是因为栈展开保证了析构函数必定执行，我们在第 10 章讲的 **RAII** 资源管理原则在此处大放异彩：只要你用 `std::unique_ptr` 或 `std::lock_guard` 等包装了资源，即便异常直接穿透了整个调用链，资源也不会泄漏一丁点！

## 万能捕获器与再次抛出

如果你不知道对方在几百层深的渊底会抛什么奇形怪状类型的异常上来，但你就是想在这一层把异常拦截并打印一句话，可以使用 `catch (...)`：

```cpp
try {
    doSomethingDangerous();
} catch (const std::exception& e) {
    // 捕获所有标准异常
    std::cerr << e.what() << "\n";
} catch (...) {
    // 终极保底：捕获剩下的所有异端类型（如 throw 1, throw "err"）
    std::cerr << "捕获到未知异常！\n";

    // 把接到的这个不知名的烫手山芋原封不动再次往更上层扔
    // 让更外头的老板去处理
    throw;
}
```

{% hint style="danger" %}
**绝对不要在析构函数中抛出异常！**
如果在栈展开的过程中（本来就是在处理一个异常了），某个局部对象的析构函数里**又**抛出了一个新的异常没有接住，C++ 面对这种“双重异常奔溃”的恶劣局面，会毫不含糊地直接暴力调用 `std::terminate()` 当场终止杀掉整个进程！
{% endhint %}
