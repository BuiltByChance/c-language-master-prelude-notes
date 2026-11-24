# 110 · 游戏菜单怎样先显示一次，选 3 才退出？

玩家要先看到菜单，才能选择新游戏、载入游戏或退出。把显示菜单、读取编号、处理选择三步放进 `do` 块，最后用 `while (choice != 3);` 决定是否再显示一次。第一次直接选 3，也能先看到一遍菜单。`choice` 在第一次判断前就由输入赋值，无需预设编号。[Microsoft Learn《do-while 语句》](https://learn.microsoft.com/zh-cn/cpp/c-language/do-while-statement-c?view=msvc-170)

## 在控制台运行

在 Visual Studio 的 C 控制台项目中，把下面的代码放进含 `main` 的 `.c` 文件，选择“调试”→“开始执行（不调试）”（`Ctrl+F5`）。每次输入一个编号并按回车。本例用文字模拟游戏过程，录屏中的游戏逻辑和存档读取都留作待实现。

```c
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void)
{
    uint32_t choice;
    do {
        puts("主菜单\n1. 新游戏\n2. 载入游戏\n3. 退出");
        puts("请选择一个选项：");
        if (scanf_s("%" SCNu32, &choice) != 1) {
            fputs("未读到整数，菜单结束。\n", stderr);
            return 1;
        }

        switch (choice) {
        case 1:
            puts("新游戏已经启动。");
            puts("游戏结束，返回主菜单。");
            break;
        case 2:
            puts("载入存档中……");
            puts("存档已经载入。");
            puts("游戏结束，返回主菜单。");
            break;
        case 3:
            puts("感谢游玩，再见！");
            break;
        default:
            puts("无效选项，请重新选择！");
        }
    } while (choice != 3);
    return 0;
}
```

**为什么选 1 后还能回到菜单？** `case 1` 的 `break` 退出最近的 `switch`，随后检查 `1 != 3`，开始下一轮。选 3 也先退出 `switch`，这次 `3 != 3` 为假，循环结束。讲师试过在 `switch` 的右花括号之后、`do` 块之内再加一条无条件 `break;`，结果选 1、2、3 都只运行一轮。那个位置的 `break` 直接退出了菜单循环。[Microsoft Learn《break 语句》](https://learn.microsoft.com/zh-cn/cpp/c-language/break-statement-c?view=msvc-170)

## 怎么判断菜单真的返回或退出了？

- 录屏中依次选 `1 → 2 → 3`，主菜单共出现三次。选 3 后虽然还打印了“返回主菜单重新选择”，随后出现的是进程退出码 0。复习程序删去了这句多余提示。判断是否返回，应看程序是否再次列出选项并等待输入。
- 录屏把“请选择一个选项”也编成了第 4 行，源码只有三个 `case`。复习程序去掉了这个编号。整理者补测输入 `4` 再输入 `3`，先看到无效选项和新一轮菜单，再看到告别提示。
- 录屏的 `scanf_s` 使用了输出宏 `PRIu32`，也没有检查返回值。整理者换成读入 `uint32_t` 的 `SCNu32`，只在返回 1 时进入 `switch`。例如输入 `x` 无法转换成整数，程序报告读取失败并结束。输入 `4` 能转换，才会进入 `default`。[C11 委员会草案 §7.8.1](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=235) · [Microsoft Learn《scanf_s 返回值》](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/scanf-s-scanf-s-l-wscanf-s-wscanf-s-l?view=msvc-170)
