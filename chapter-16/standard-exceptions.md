# 16.2 标准异常类

在 C++ 中，`throw` 后面可以跟任意类型（你可以 `throw 42;` 或者 `throw "ERROR";`），但这并不是一种优秀的工程实践。

C++ 标准库在 `<stdexcept>` 中规定了一套非常完善的异常类继承体系。在现代 C++ 开发中，**只建议抛出继承自 `std::exception` 的类对象**。

## 异常基类 std::exception

它是所有标准异常的太爷爷祖宗。它仅仅暴露了一个极其重要的虚函数：

```cpp
virtual const char* what() const noexcept;
```

这个函数返回一个以 null 结尾的 C 风格字符串，用于描述这个错误具体长什么样（通常就是我们在下面那些派生类构造时传进去的字符串提示）。由于存在这个基类，我们可以通过多态，只要 `catch(const std::exception& e)` 就能接住所有标准库或规范开发的异常对象，然后调用 `e.what()` 打印。

## 两大常见派生派系

标准库将异常大体划分为两拨：
1. **逻辑错误（Logic Errors）**：程序员的锅，本可以在代码写得仔细点（比如做好提前防范判定）就能完全避免的错误。
2. **运行时错误（Runtime Errors）**：无法预测的外部环境锅，通常是程序无法防范的突发事件（如网络断线，读文件被系统拔出 U 盘）。

### 派系一：`std::logic_error`

这是基类衍生出的第一个大类，常用的实装异常类有：
- `std::out_of_range`：最常用！当你用 `std::vector::at(99)` 越界访问，由于超出了范围，容器就会扔出这个异常。（注意原生的原生数组 `[99]` 不抛这个，直接未定义行为内存越界）。
- `std::invalid_argument`：比如你在开发一个函数要求入参必须是非负数，别人传了负数，你就可以 `throw std::invalid_argument("年龄不能为负")`。
- `std::length_error`：如果试图创建一个超出该容器理论支持最大上限（极度巨大的数）的 `string` 就会报错这种。

### 派系二：`std::runtime_error`

常用的包括：
- `std::overflow_error` / `std::underflow_error`：数值上浮或下溢（主要是数学库或者类型转换可能使用）。

除此之外，直接继承自最上层基类但我们也会非常常遇见的是由核心语言关键字触发的：
- `std::bad_alloc`：在用 `new` 申请内存因为内存不足被操作系统拒绝时抛出！
- `std::bad_cast`：在使用 `dynamic_cast` 对引用做多态强行向下转型失败时抛出。

## 自定义异常类

在实际业务开发中，光用标准库那几个常常难以满足我们需要夹带额外“私货（如 HTTP 错误码）”的需求，标准的做法是**自己写一个类继承标准异常类**。

```cpp
#include <iostream>
#include <stdexcept>
#include <string>

// 自定义业务级异常，继承自 std::runtime_error
class NetworkException : public std::runtime_error {
private:
    int errorCode;

public:
    // 构造时不仅传入必须的报错信息给父类，还要记录下暗藏的私货 errorCode
    NetworkException(const std::string& msg, int code)
        : std::runtime_error(msg), errorCode(code) {}

    // 暴露私用功能
    int getErrorCode() const { return errorCode; }
};

int main() {
    try {
        // 遇到丢包
        throw NetworkException("Connection Timeout", 504);

    } catch (const NetworkException& e) {  // 专门捕获网络异常
        std::cerr << "[Network ERROR " << e.getErrorCode() << "] " << e.what() << "\n";

    } catch (const std::exception& e) {    // 捕获其余所有不慎漏过的符合标准的异常
        std::cerr << "[General ERROR] " << e.what() << "\n";
    }

    return 0;
}
```

{% hint style="info" %}
**异常捕获顺序规则**
如果你写了多个连续的 `catch` 块（如上一段代码），**必须把最精细派生的子异常类放在前面，把大基类（如 `std::exception` 或万能 `...`）放在最后兜底！**
因为判明逻辑是从上倒下匹配的，大基类依靠多态能接住所有的儿子，这会导致后面的精细儿子捕获块永远失效！
{% endhint %}