# 109 · 条件一开始就为假，while 和 do-while 各跑几圈？

选循环时，先问**第一轮是否也要满足条件**。如果目标是 0 圈就应原地不动，用 `while`，它在进入循环体前判断。若约定先跑一圈、再决定是否继续，用 `do-while`，它在一轮结束后判断。[C 的 while 语句](https://learn.microsoft.com/zh-cn/cpp/c-language/while-statement-c?view=msvc-170) · [C 的 do-while 语句](https://learn.microsoft.com/zh-cn/cpp/c-language/do-while-statement-c?view=msvc-170)

## 用同一段计数代码比较

在 Visual Studio 的 C 控制台项目中，将下面的代码放进含 `main` 的 `.c` 文件，从“调试”菜单选“开始执行（不调试）”（`Ctrl+F5`）。`total_laps` 是目标圈数，`current_lap` 是实际已跑圈数，初始为 0。

```c
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>
#include <stdbool.h>

int main(void)
{
    uint32_t total_laps = 10;
    uint32_t current_lap = 0;

    puts("罚跑开始");
    do {
        current_lap++;
        printf("跑步者完成第%" PRIu32 "圈\n", current_lap);
    } while (current_lap < total_laps);
    return 0;
}
```

先自增再打印，所以第一行圈数是 1。打印第 10 圈后才判断 `10 < 10`，结果为假，程序结束。`do-while` 末尾的分号属于循环语法。

换成 `while` 时，用下面这段**替换**整个 `do-while`。每次重新运行，让计数都从 0 开始。

```c
while (current_lap < total_laps) {
    current_lap++;
    printf("跑步者完成第%" PRIu32 "圈\n", current_lap);
}
```

| 分别运行两个版本 | `while` 输出的圈数 | `do-while` 输出的圈数 |
| --- | --- | --- |
| 录屏实测，目标为 10 | 第 1 至第 10 圈 | 第 1 至第 10 圈 |
| 录屏实测，把各自括号中的比较式改成 `false` | 没有圈数，只有循环外的“罚跑开始” | 第 1 圈 |
| 整理者补测，保留比较式，把 `total_laps` 改成 0 | 无圈数行，入口处的 `0 < 0` 为假 | 第 1 圈，随后检查的 `1 < 0` 为假 |

## 怎样核对输出

- **核对循环次数**时，看循环体内打印的 `current_lap`。录屏中，`do-while` 条件改成 `false` 后只跑了 1 圈，末行却报告完成 10 圈。那一行在循环外，打印的是目标 `total_laps`。讲师把实参改为 `current_lap` 后末行显示 1，随后删去了这条重复输出。
- **遇到 Visual Studio 警告 C6271**（传给 `printf` 的实参多了）时，检查每个待打印的圈数是否有对应的格式。录屏中 `PRIu32` 展开为 `"u"`，字符串里漏掉 `%` 就只剩普通字母。写成 `"%" PRIu32` 才能按无符号十进制打印 `uint32_t`。[C11 委员会草案 §7.8.1](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=235)
