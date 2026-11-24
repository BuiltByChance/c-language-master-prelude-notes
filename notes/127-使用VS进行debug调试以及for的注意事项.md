# 127 · 怎样用 Visual Studio 找到嵌套循环第一次走错的位置？

先写出一组可核对的编号。课程案例有 3 张桌子，每桌 4 个座位。外层从桌 1 到桌 3，内层从座位 1 到座位 4；从 `(桌 1, 座位 4)` 往后应到 `(桌 2, 座位 1)`。只打印桌号先核对三行，再加座号核对 12 组，能分清哪层循环有错。录屏第一次运行完整打印到桌 3 座 4，进程以 0 退出。座位分配业务留在注释里。

下面是整理者按录屏缩短的练习，可放入 Visual Studio 控制台项目的 `.c` 文件。

```c
#include <stdio.h>

int main(void)
{
    const int NUM_TABLES = 3;
    const int SEATS_PER_TABLE = 4;
    for (int table_num = 1; table_num <= NUM_TABLES; ++table_num) {
        printf("桌 %d\n", table_num);
        for (int seat_num = 1; seat_num <= SEATS_PER_TABLE; ++seat_num) {
            printf("  桌 %d，座位 %d\n", table_num, seat_num);
        }
    }
    return 0;
}
```

1. 在两个 `printf` 左侧的编辑器边栏各单击一次，看到实心红点。选工具栏 `Debug` 配置，按 **F5**（`Debug → Start Debugging`）启动。**Ctrl+F5** 是不调试运行，检查断点用 F5。[Microsoft 的断点文档](https://learn.microsoft.com/zh-cn/visualstudio/debugger/using-breakpoints?view=visualstudio)
2. 黄色箭头标出下一条将执行的语句。暂停后从 `Debug → Windows → Locals` 打开局部变量，读 `Name` 与 `Value`。停在外层输出语句时，`table_num` 已初始化；停在内层输出语句时，`seat_num` 也已初始化。录屏曾把断点打在 `for` 行，在 `Autos` 窗口中，桌号初始化前曾显示 0，内层初始化前座号曾显示 688，不能当作 C 保证的初值。窗口中变红的值表示自上次停下以来发生了变化。[Microsoft 的变量窗口说明](https://learn.microsoft.com/zh-cn/visualstudio/debugger/autos-and-locals-windows?view=visualstudio)
3. 按 **F5（Continue）** 到下个命中的断点，核对桌号保持 1 时座号依次为 1、2、3、4。若要看当前输出执行后控制台多了哪一行，用 **F10（Step Over，逐过程）**。**F11（Step Into，逐语句）** 遇到有可用调试代码的函数调用时可以进入函数。检查完用 `Debug → Stop Debugging` 停止；要清除整组断点用 `Debug → Delete All Breakpoints`。[Microsoft 的单步执行说明](https://learn.microsoft.com/zh-cn/visualstudio/debugger/navigating-through-code-with-the-debugger?view=visualstudio)

若第一张桌从座位 2 开始，查内层初值。每桌都少座位 4，查是否误写 `< SEATS_PER_TABLE`。换桌后座号没有回到 1，查初始化是否放在每次进入内层时执行的位置。不用调试器时，可在疑似出错的条件前、分支内分别打印位置标签和两个编号，第一处与预计不符的输出能缩小排查范围。
