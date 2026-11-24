# 159 · 怎样声明无参函数，让编译器发现多传的实参？

`void greet(void);` 里，名字前的 `void` 表示函数不返回值，括号内单独的 `void` 表示参数个数为零。声明和定义可以都写 `(void)`，调用时只写 `greet();`。

```c
#include <stdio.h>

void greet(void);

int main(void)
{
    greet();
    return 0;
}

void greet(void)
{
    puts("Hello");
}
```

这份整理者复习示例会打印一行 `Hello`。若把调用写成 `greet(age)`，可见的原型已明确参数个数，编译器应诊断实参多余。

### 空括号的含义取决于 C 版本

- **C17 及以前**，只写声明 `void greet();` 没有告诉调用方参数个数和类型。如果定义写成 `void greet() { /* ... */ }`，该定义实际有零个参数，通过旧式声明多传实参会导致未定义行为。[WG14 C11 草案 §6.7.6.3 第 14 段](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n1570.pdf#page=152)
- **C23** 规定空括号与 `(void)` 都表示零参数。要让同一个头文件在这两类标准模式下都能检查无参调用，写 `void greet(void);`。[WG14 C23 草案 §6.7.6.3 第 13 段](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n3096.pdf#page=147)

### 在 Visual Studio 分清运行结果与生成警告

录屏里 `age = 20`，给实际无参的函数传一个或两个整数都曾在控制台看到 `Hello`。改成显式 `(void)` 后，多传 `age` 时 MSVC 仍生成了可执行文件，同时报告 **C4087**。打开“视图 → 输出”，将“显示输出来源”切换为“生成”，按警告号查看对应调用行。如果输出只写项目已是最新状态，可用“生成 → 重新生成解决方案”重看完整编译诊断。录屏另一个警告 **C4101** 指局部变量 `aaa` 从未使用，删掉它不会修复 `greet(age)`。[Microsoft 对 C4087 的说明](https://learn.microsoft.com/en-us/cpp/error-messages/compiler-warnings/compiler-warning-level-1-c4087?view=msvc-170)

课程最后用数组作类比。长度为 5 的数组允许的元素下标是 0 到 4，访问下标 5 的元素行为未定义。讲师只讲了这个例子，没有展示一次越界操作的运行结果。
