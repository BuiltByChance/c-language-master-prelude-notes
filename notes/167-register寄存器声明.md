# 167 · register 能保证循环计数器放进 CPU 寄存器吗？

CPU 寄存器可以保存参与运算的数据、地址或处理器状态，寄存器的种类和数量由架构决定。在 C 中，`register int counter;` 是给实现的访问速度建议，编译器决定是否、怎样采纳；普通 `int counter;` 也可能被优化到寄存器。两者在这个循环里都是自动存储期的局部整数。[WG14 C11 草案 §6.7.1 第 6 段](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n1570.pdf#page=128)

```c
#include <stdio.h>

int main(void)
{
    register int counter;
    for (counter = 0; counter < 10; ++counter) {
        printf("%d ", counter);
    }
    putchar('\n');
    return 0;
}
```

整理者把课堂的循环放进 `main` 便于复习。计数器先设 0，每次打印后加 1，等于 10 时退出；录屏实际输出 **0 1 2 3 4 5 6 7 8 9**，退出码 0。第一次生成失败是上节遗留的 `helper.obj` 引用了未提供的 `g_static_var`；移去旧文件后，这个循环才运行。课中没有查看机器指令，也没有比较耗时，控制台输出只能验证循环结果。

选声明时有一个确定差异：C 禁止用一元 `&` 取得声明为 `register` 的对象的地址，即使编译器最后把它放在内存里，这条约束仍有效。因此要用 `scanf("%d", &counter)` 向计数器写入整数时，应声明普通的 `int counter;`。若关心循环速度，应在目标平台比较相同优化设置下的指令或耗时，计时区中逐次 `printf` 还会引入输出开销。[WG14 C23 草案 §6.5.3.2 第 1 段](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n3096.pdf#page=99)
