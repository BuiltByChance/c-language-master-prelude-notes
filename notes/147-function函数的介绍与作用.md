# 147 · 怎样把重复的输出提取成函数？

C 函数是可以按名称调用的一段操作。它可以把计算结果返回给调用者，也可以只产生外部可见的效果。例如，`print_line` 在终端打印一条分隔线，返回类型写成 `void`。一次调用执行一次函数体，执行完后回到调用处的下一条语句。

录屏把 `main` 中反复出现的两段代码分别提取出来：`print_line` 打印一行横线，`loop_i` 用循环打印五行 `iiiii`。两个函数都定义在 `main` 前面，`main` 中保留原来的调用顺序。下面是整理者按录屏顺序写出的复习版，`(void)` 明确表示不接收参数。

```c
#include <stdio.h>

void print_line(void)
{
    puts("----------------------");
}

void loop_i(void)
{
    for (int i = 0; i < 5; ++i) {
        puts("iiiii");
    }
}

int main(void)
{
    print_line();
    loop_i();
    print_line();
    loop_i();
    loop_i();
    print_line();
    loop_i();
    loop_i();
    return 0;
}
```

**检查提取是否改变了行为。** 一共调用三次 `print_line`、五次 `loop_i`。三个分隔线下面应依次有 5、10、10 行 `iiiii`，录屏重构后的程序显示这三组，并以 0 退出。若准备把每组基础输出从五行改为三行，只需改 `loop_i` 的循环上界，五处调用会一起生效。
