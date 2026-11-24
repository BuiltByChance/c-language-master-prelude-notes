# 112 · 为什么输入负数用 continue，输入 0 用 break？

连续读取整数，只累加正数。读到负数时，后面的数还要处理，所以用 `continue` 跳过本轮的 `sum += number;`，回到 `while (1)` 再读一次。读到 `0` 时，本次求和已经结束，用 `break` 离开循环，到循环外显示总和。[Microsoft Learn《continue》](https://learn.microsoft.com/zh-cn/cpp/c-language/continue-statement-c?view=msvc-170) · [《break》](https://learn.microsoft.com/zh-cn/cpp/c-language/break-statement-c?view=msvc-170)

**沿录屏里的 `-1` 走一遍。** `-1 == 0` 为假，跳过 `break`。`-1 < 0` 为真，执行 `continue`，本轮不会走到加法。换成 `1`，两个条件都为假，才执行 `sum += number;`。因此，若把“跳过负数”写成 `if (number <= 0) continue;` 并放在零值检查前面，输入 `0` 也会回去读下一个数，无法结算。

每轮只有一次累加时，保留零值退出判断，随后写 `if (number > 0) sum += number;` 也能得到相同结果。若后面还要统计正数个数，提前跳过负数，累加和计数就能共用同一个筛选条件。

## 运行并核对结果

录屏的 `puts` 提示输入 0 后会显示总和，但片尾循环后直接是 `return 0;`，全程没有运行程序。下面是整理者补全的版本。在 Visual Studio 的 C 控制台项目中，把这段代码放进含 `main` 的 `.c` 文件，按 `Ctrl+F5` 运行。

```c
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void)
{
    uint32_t sum = 0;
    int32_t number;
    puts("输入整数，负数跳过，0 结束。");

    while (1) {
        if (scanf_s("%" SCNd32, &number) != 1) {
            fputs("未读到整数，本次求和结束。\n", stderr);
            return 1;
        }
        if (number == 0) {
            break;
        }
        if (number < 0) {
            continue;
        }
        if ((uint32_t)number > UINT32_MAX - sum) {
            fputs("总和超出 32 位无符号整数范围。\n", stderr);
            return 1;
        }
        sum += (uint32_t)number;
    }

    printf("总和 = %" PRIu32 "\n", sum);
    return 0;
}
```

输入 `4 -1 1 0 9` 并回车，实测输出 `总和 = 5`。`-1` 后面的 `1` 被加进来，证明负数只跳过一轮。`0` 后面的 `9` 没被加进来，证明零结束了整个循环。直接输入 `0` 则输出 `总和 = 0`。

每项输入限于 `int32_t` 可表示的整数。`SCNd32` 用于读有符号的 `number`，`PRIu32` 用于显示无符号的 `sum`。`scanf_s` 返回 1 才有新值可用，输入 `q` 会报错结束。正数超过 `UINT32_MAX - sum` 表示总和剩余容量不足，程序在加法前报错。[Microsoft Learn《scanf_s 返回值》](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/scanf-s-scanf-s-l-wscanf-s-wscanf-s-l?view=msvc-170) · [C11 委员会草案 §7.8.1](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=235)
