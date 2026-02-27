# Summary

* [简介](README.md)

## 第一部分：入门准备

* [第一章 认识 C++](chapter-01/README.md)
  * [1.1 C++ 的历史与发展](chapter-01/history.md)
  * [1.2 开发环境搭建](chapter-01/environment-setup.md)
  * [1.3 第一个程序：Hello World](chapter-01/hello-world.md)
  * [1.4 编译与运行](chapter-01/compile-and-run.md)

## 第二部分：语言基础

* [第二章 基本概念](chapter-02/README.md)
  * [2.1 变量与数据类型](chapter-02/variables-and-types.md)
  * [2.2 运算符与表达式](chapter-02/operators.md)
  * [2.3 输入与输出](chapter-02/input-output.md)
  * [2.4 类型转换](chapter-02/type-conversion.md)
  * [2.5 补充：理解位与字节](chapter-02/storage-basics.md)
* [第三章 流程控制](chapter-03/README.md)
  * [3.1 条件语句](chapter-03/conditional.md)
  * [3.2 循环语句](chapter-03/loop.md)
  * [3.3 跳转语句](chapter-03/jump.md)
* [第四章 函数](chapter-04/README.md)
  * [4.1 函数的定义与调用](chapter-04/definition-and-call.md)
  * [4.2 参数传递](chapter-04/parameter-passing.md)
  * [4.3 函数重载](chapter-04/overloading.md)
  * [4.4 递归](chapter-04/recursion.md)
  * [4.5 内联函数与 constexpr](chapter-04/inline-and-constexpr.md)
* [第五章 数组与字符串](chapter-05/README.md)
  * [5.1 数组](chapter-05/array.md)
  * [5.2 C 风格字符串](chapter-05/c-string.md)
  * [5.3 std::string](chapter-05/std-string.md)
  * [5.4 std::array 与 std::vector 初步](chapter-05/array-and-vector.md)

## 第三部分：面向对象编程

* [第六章 类与对象](chapter-06/README.md)
  * [6.1 类的定义](chapter-06/class-definition.md)
  * [6.2 构造函数与析构函数](chapter-06/constructor-destructor.md)
  * [6.3 访问控制](chapter-06/access-control.md)
  * [6.4 this 指针](chapter-06/this-pointer.md)
  * [6.5 静态成员](chapter-06/static-member.md)
* [第七章 继承与多态](chapter-07/README.md)
  * [7.1 继承](chapter-07/inheritance.md)
  * [7.2 虚函数与多态](chapter-07/virtual-function.md)
  * [7.3 抽象类与纯虚函数](chapter-07/abstract-class.md)
  * [7.4 多重继承](chapter-07/multiple-inheritance.md)
* [第八章 运算符重载](chapter-08/README.md)
  * [8.1 重载的基本规则](chapter-08/basic-rules.md)
  * [8.2 常见运算符重载](chapter-08/common-operators.md)
  * [8.3 类型转换运算符](chapter-08/conversion-operator.md)

## 第四部分：内存与资源管理

* [第九章 指针与引用](chapter-09/README.md)
  * [9.1 指针基础](chapter-09/pointer-basics.md)
  * [9.2 引用](chapter-09/reference.md)
  * [9.3 动态内存分配](chapter-09/dynamic-memory.md)
  * [9.4 const 与指针](chapter-09/const-and-pointer.md)
* [第十章 智能指针与资源管理](chapter-10/README.md)
  * [10.1 RAII 原则](chapter-10/raii.md)
  * [10.2 unique_ptr](chapter-10/unique-ptr.md)
  * [10.3 shared_ptr 与 weak_ptr](chapter-10/shared-and-weak-ptr.md)
  * [10.4 移动语义与右值引用](chapter-10/move-semantics.md)

## 第五部分：泛型编程与标准库

* [第十一章 模板](chapter-11/README.md)
  * [11.1 函数模板](chapter-11/function-template.md)
  * [11.2 类模板](chapter-11/class-template.md)
  * [11.3 模板特化](chapter-11/template-specialization.md)
  * [11.4 变参模板](chapter-11/variadic-template.md)
* [第十二章 STL 容器](chapter-12/README.md)
  * [12.1 序列容器](chapter-12/sequence-containers.md)
  * [12.2 关联容器](chapter-12/associative-containers.md)
  * [12.3 无序容器](chapter-12/unordered-containers.md)
  * [12.4 容器适配器](chapter-12/container-adapters.md)
* [第十三章 STL 算法与迭代器](chapter-13/README.md)
  * [13.1 迭代器](chapter-13/iterator.md)
  * [13.2 常用算法](chapter-13/common-algorithms.md)
  * [13.3 Lambda 表达式](chapter-13/lambda.md)

## 第六部分：现代 C++ 特性

* [第十四章 C++11/14 核心特性](chapter-14/README.md)
  * [14.1 auto 与 decltype](chapter-14/auto-and-decltype.md)
  * [14.2 范围 for 循环](chapter-14/range-for.md)
  * [14.3 初始化列表](chapter-14/initializer-list.md)
  * [14.4 空指针 nullptr](chapter-14/nullptr.md)
  * [14.5 枚举类](chapter-14/enum-class.md)
* [第十五章 C++17 重要特性](chapter-15/README.md)
  * [15.1 结构化绑定](chapter-15/structured-bindings.md)
  * [15.2 std::optional / std::variant / std::any](chapter-15/optional-variant-any.md)
  * [15.3 if constexpr](chapter-15/if-constexpr.md)
  * [15.4 std::filesystem](chapter-15/filesystem.md)
  * [15.5 折叠表达式](chapter-15/fold-expressions.md)

## 第七部分：工程实践

* [第十六章 异常处理](chapter-16/README.md)
  * [16.1 异常机制](chapter-16/exception-mechanism.md)
  * [16.2 标准异常类](chapter-16/standard-exceptions.md)
  * [16.3 异常安全](chapter-16/exception-safety.md)
* [第十七章 文件与 IO](chapter-17/README.md)
  * [17.1 文件流操作](chapter-17/file-stream.md)
  * [17.2 字符串流](chapter-17/string-stream.md)
  * [17.3 格式化输出](chapter-17/formatting.md)
* [第十八章 编译与构建](chapter-18/README.md)
  * [18.1 编译过程详解](chapter-18/compilation-process.md)
  * [18.2 头文件与源文件组织](chapter-18/header-and-source.md)
  * [18.3 CMake 基础](chapter-18/cmake-basics.md)
