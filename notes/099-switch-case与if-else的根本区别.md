# 099 · if 判断真假，switch 匹配取值：星期编号该怎样分支？

按一个编号分派操作，比如 4 对应星期四、5 对应星期五，用 `switch` 可以直接列出编号。判断负数、零、正数，或检查 `x > 5 && x < 7` 这样的范围，用 `if / else if / else` 能把每个条件写清楚。

**同一个 5 在两种语句里意味着什么？** `if (day_of_week)` 只检查它是否非零，4、5 和 −1 都会进入分支。只想匹配星期五，要写 `if (day_of_week == 5)`，或者在 `switch (day_of_week)` 里写 `case 5:`。C 的 `if` 可以检查整数、浮点数或指针等标量表达式，无需先把结果存成 `bool`。[C11 委员会草案 §6.8.4](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=166)

下面是整理者补全输出的复习程序。课堂上的分支体主要是空块或注释。

```c
#include <stdio.h>

int main(void) {
    int day_of_week = 5;

    if (day_of_week) {
        puts("编号非零");
    }

    switch (day_of_week) {
    case 4:
        puts("星期四");
        break;
    case 5:
        puts("星期五");
        break;
    }

    puts("判断结束");
    return 0;
}
```

**什么时候可以省略兜底分支？** 希望未列出的编号直接继续往下走，就可以像上例一样省略 `default`。每个 `break` 使已匹配的分支退出 `switch`。如果要报告“未知编号”，就在 `default:` 里输出提示。`if` 链也只在需要统一处理剩余情况时才写末尾的 `else`。[Microsoft Learn：C 的 switch 语句](https://learn.microsoft.com/en-us/cpp/c-language/switch-statement-c?view=msvc-170)

整理者实测，将初值改为 `5` 或 `4`，先输出“编号非零”，再输出对应星期，最后输出“判断结束”。改为 `-1`，只输出“编号非零”和“判断结束”。改为 `0`，只输出“判断结束”。

**看到 `expression must have integral type`，检查哪个表达式？** 先看 `switch (...)` 括号内的类型。它必须是整数类型，`char`、`int`、枚举都可以。再看 `case` 后的标签，它必须是整数常量表达式，同一 `switch` 中各标签转换后的值不能重复。课堂把星期变量改为 `float day_of_week = 5.0f;` 后，`if (day_of_week == 5)` 仍然合法，`switch (day_of_week)` 则标红。[C11 委员会草案 §6.8.4.2](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=167)

课堂中的 `switch ("Frank")` 和 `switch (str)`（`str` 是 `char *`）也触发了整数类型诊断。整理者补充的做法是包含 `<string.h>`，用 `if (strcmp(str, "Frank") == 0)` 判断两个有效字符串的内容是否相同。[Microsoft Learn：strcmp](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strcmp-wcscmp-mbscmp?view=msvc-170)

**性能不能从分支数直接推出。** 源代码匹配到 `case 5` 后会从那里开始执行。底层怎样找到它由编译器决定，可能生成一串条件分支，也可能查表。课堂没有查看生成的指令或计时。如果速度会影响选型，要在同一目标平台、相同优化设置和同一组输入下测量两种等价实现。[LLVM 的 switch 指令说明](https://llvm.org/docs/LangRef.html#switch-instruction)
