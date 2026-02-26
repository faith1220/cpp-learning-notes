# 3.3 跳转语句

跳转语句用于在循环或函数中改变正常的执行流程。C++ 提供了四种跳转语句：`break`、`continue`、`return` 和 `goto`。

## break

`break` 用于**立即终止**当前所在的循环或 `switch` 语句，程序跳转到循环体之后的第一条语句继续执行。

```cpp
// 找到第一个能被 7 整除的数
for (int i = 1; i <= 100; ++i) {
    if (i % 7 == 0) {
        std::cout << "第一个能被 7 整除的正整数是 " << i << std::endl;
        break;   // 找到后立即退出循环
    }
}
// 输出：第一个能被 7 整除的正整数是 7
```

在嵌套循环中，`break` 只跳出**最内层**的循环：

```cpp
for (int i = 0; i < 3; ++i) {
    for (int j = 0; j < 3; ++j) {
        if (j == 1) {
            break;   // 只跳出内层循环
        }
        std::cout << "i=" << i << " j=" << j << std::endl;
    }
}
// 输出：
// i=0 j=0
// i=1 j=0
// i=2 j=0
```

如果需要跳出多层循环，通常的做法是使用标志变量或将循环逻辑提取到独立函数中，用 `return` 退出。

## continue

`continue` 用于**跳过当前迭代的剩余部分**，直接进入下一次迭代。

```cpp
// 输出 1 到 10 中所有奇数
for (int i = 1; i <= 10; ++i) {
    if (i % 2 == 0) {
        continue;   // 跳过偶数
    }
    std::cout << i << " ";
}
std::cout << std::endl;
// 输出：1 3 5 7 9
```

在 `for` 循环中，`continue` 会跳到迭代表达式（如 `++i`）继续执行。在 `while` 和 `do-while` 中，`continue` 会跳到条件判断处。

{% hint style="warning" %}
在 `while` 循环中使用 `continue` 时要特别注意：如果循环变量的更新语句写在 `continue` 之后，`continue` 会跳过更新，可能导致死循环。

```cpp
int i = 0;
while (i < 10) {
    if (i == 5) {
        continue;   // 跳过了 ++i，i 永远等于 5，死循环
    }
    std::cout << i << " ";
    ++i;
}
```

应将 `++i` 放在 `continue` 之前，或改用 `for` 循环避免此问题。
{% endhint %}

## return

`return` 用于**结束当前函数的执行**，并可选地返回一个值给调用者。

```cpp
int findFirst(int arr[], int size, int target) {
    for (int i = 0; i < size; ++i) {
        if (arr[i] == target) {
            return i;    // 找到目标，立即返回其下标
        }
    }
    return -1;   // 未找到，返回 -1
}
```

在 `main` 函数中，`return 0` 表示程序正常结束，`return` 非零值表示出错。

```cpp
int main() {
    // ...
    if (errorOccurred) {
        return 1;   // 异常退出
    }
    return 0;       // 正常退出
}
```

`return` 会终止整个函数，不仅仅是跳出循环。如果在嵌套循环中调用 `return`，所有循环层级都会被跳出。

## goto

`goto` 可以无条件跳转到同一函数内的某个**标签**（Label）处：

```cpp
for (int i = 0; i < 10; ++i) {
    for (int j = 0; j < 10; ++j) {
        if (i + j == 15) {
            goto found;   // 跳出所有循环
        }
    }
}
found:
std::cout << "找到了" << std::endl;
```

{% hint style="danger" %}
`goto` 在现代 C++ 中**极少使用**。它会破坏代码的结构化，使程序流程难以追踪和维护。几乎所有需要 `goto` 的场景都可以用其他方式替代（如标志变量、函数封装、异常等）。唯一被广泛接受的用途是跳出多层嵌套循环，但即使这种情况也有更好的替代方案。除非有明确的理由，否则不要使用 `goto`。
{% endhint %}

## 总结

| 语句 | 作用 | 适用范围 |
|------|------|----------|
| `break` | 终止当前循环或 `switch` | 循环体、`switch` |
| `continue` | 跳过当前迭代，进入下一次迭代 | 循环体 |
| `return` | 终止当前函数并返回值 | 任何位置 |
| `goto` | 无条件跳转到标签处 | 同一函数内（不推荐使用） |
