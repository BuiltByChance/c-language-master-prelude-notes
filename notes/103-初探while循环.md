# 103 · while 为什么打印完第 10 圈就停，漏掉自增却一直打印 0？

`current_lap` 表示**已经跑完的圈数**，所以从 0 开始。目标是跑满 10 圈，`while` 括号里就填“还要继续跑”的条件 `current_lap < TOTAL_LAPS`。达到目标时，这个条件自然变为假。

## 写出程序并核对输出

下面按录屏整理，可以放进 Visual Studio 控制台项目的 `.c` 源文件。整理者补充的运行方法是从“调试”菜单选“开始执行（不调试）”，默认快捷键为 `Ctrl+F5`。[Visual Studio 运行命令](https://learn.microsoft.com/zh-cn/visualstudio/ide/building-and-cleaning-projects-and-solutions-in-visual-studio?view=visualstudio)

```c
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void)
{
    const uint32_t TOTAL_LAPS = 10;
    uint32_t current_lap = 0;

    puts("开始罚跑");
    while (current_lap < TOTAL_LAPS) {
        current_lap++;
        printf("跑步的人完成第 %" PRIu32 " 圈。\n", current_lap);
    }
    return 0;
}
```

`const` 让目标圈数保持不变。打印 `uint32_t` 类型的圈数时，`%` 和 `<inttypes.h>` 提供的 `PRIu32` 合起来才是完整的无符号十进制格式。录屏里讲师补上了漏写的 `%`。[C 工作组草案 §7.8.1](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf)

运行时先出现一次“开始罚跑”，然后依次报告第 1 到第 10 圈。看最后两次条件判断，注意打印发生在加一之后。

| 检查条件时已跑完 | 判断 | 接下来发生什么 |
| --- | --- | --- |
| 9 圈 | `9 < 10` 为真 | 进入循环体，加到 10，打印第 10 圈 |
| 10 圈 | `10 < 10` 为假 | 跳到右花括号之后，执行 `return 0;` |

循环体执行一次叫一次迭代。这里迭代 10 次，判断条件 11 次。讲师用初始化、条件判断、循环体、更新、终止五步走读这个计数循环。最后的“终止”就是回到同一个 `while` 条件时发现它已为假。若第一次检查就为假，循环体执行 0 次。[Microsoft Learn《while 语句（C）》](https://learn.microsoft.com/zh-cn/cpp/c-language/while-statement-c?view=msvc-170)

## 改一行，输出为什么变了？

| 讲师实际运行的改动 | 控制台结果 | 原因 |
| --- | --- | --- |
| 注释掉 `current_lap++;` | 反复报告第 0 圈，程序持续运行 | `printf` 只读取圈数。回到条件处时仍是 `0 < 10`，条件一直为真 |
| 把自增改为 `current_lap += 2;` | 依次报告 2、4、6、8、10，共五行圈数 | 每轮先增加两圈再打印，到 10 后下一次检查为假 |

整理者补了一道边界练习。保留每轮加一，把条件改成 `current_lap <= TOTAL_LAPS`。当已完成 10 圈时，`10 <= 10` 仍为真，还会进入循环体并打印第 11 圈。遇到“多跑一次”，先把目标值代入条件，再看本轮是在打印前还是打印后更新。
