# 17.2 字符串流

上一节我们把“流操作”建立在了硬盘上，形成 `fstream`。如果我们将这种“流”的思想引水灌溉到一条存在于内存中的**普通字符串**之上呢？

那就是定义在 `<sstream>` 中的 **字符串流（String Stream）**：`std::stringstream`。

我们可以把它想象成一个内置在内存里的“隐藏记事本缓冲区”。你可以用 `<<` 不断往里挤塞各种奇形怪状类型的数据（它会自动转成文字拼接），也可以用 `>>` 像抽取提款机一样按照指定的强类型变量槽位从中提取转化出数据来。

## 最强形态的拼接法

在 C++ 中，如果我们想用原生的底层做法把数字、浮点数以及字符串缝合到一起，异常麻烦（你可能需要去搜一搜 `std::to_string` 等等琐碎工具函数）。有了字符串流，一切如同探囊取物！

```cpp
#include <iostream>
#include <sstream> // 必须包含
#include <string>

int main() {
    std::stringstream ss; // 建一块内存空画板

    int apples = 5;
    double price = 1.25;
    std::string name = "Alice";

    // 疯狂作画拼接：流操作符会自动执行隐式的转码翻译（比如把 int 变文本字符 5）
    ss << name << " 买了 " << apples << " 个苹果，每个 $" << price << "。";

    // 画完之后，通过 .str() 接口一次性取走整幅画的字符串副本
    std::string final_sentence = ss.str();

    std::cout << final_sentence << "\n";
    // 输出: Alice 买了 5 个苹果，每个 $1.25。

    return 0;
}
```

## 神奇的数据抽取（类型转换利器）

如果我们现在拿到了一段极其杂乱包含数字和符号的日志文本。怎么优雅地自动把里面隐含着的不同强类型切分拨拉出来？

字符串流内部的 `>>` 抽取算符同样懂得见缝插针（遇到空格或回车便会停下完成一次收割转换）：

```cpp
#include <iostream>
#include <sstream>
#include <string>

int main() {
    // 这次我们在构造时，一开始就直接塞一条硬菜预备在它肚子里等着被拿走
    std::string logLine = "Warning 404 UserNotFound 2023-11-20";
    std::stringstream ss(logLine);

    // 准备容器接货
    std::string level;
    int errorCode;
    std::string extraMsg;
    std::string date;

    // 开始接力水泵！
    // 第一抽给字符串，吸走 "Warning"，撞到空格停下。
    // 第二抽点名要整数，把文本 "404" 变异翻译成了真实数字！
    ss >> level >> errorCode >> extraMsg >> date;

    std::cout << "分析结论: 级别[" << level
              << "] 代码(" << errorCode << ") \n";
    // errorCode 此时是真正的 int(404)，你可以拿它去做如 > 400 等数学逻辑比较了！

    return 0;
}
```

## 重置流以便循环复用

如果你在处理大批量的日志（比如使用一个循环）。如果在每一次大括号循环体里反复生造再立刻抛弃一个全新的 `std::stringstream ss;`，这是会有较重的运行期分配系统开销性能损害的。

正确做法是在循环外创立它，在循环内去“大清洗重置”以便洗旧用新。

```cpp
std::stringstream ss;

for(int i = 0; i < 100; ++i) {
    // 1. 真真正正物理抽调光它内部保留的旧文本块字句
    ss.str("");

    // 2. 将它因为上次抽得榨干到底触发了 EOF(结束标志位)或出错标定的警报状态灯进行强行消除清零
    ss.clear();

    // 洗刷完成，此时它崭新且干净！
    ss << "Number_" << i;
    // ...
}
```

{% hint style="danger" %}
初学者极易犯的错就在上边这个清洗操作！
大部分人只调用了 `ss.str("")` 以为清空了字符就没事了。如果没有 `ss.clear()` 恢复流的读取绿灯就绪指示位，下一次你试图用 `>>` 从里面拔数据或者强塞的时候由于它的状态还是死局，所有操作全会被假死拦截屏蔽拒之门外！
{% endhint %}