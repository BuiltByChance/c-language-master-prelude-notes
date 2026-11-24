# 158 · 怎样只记前两项，用循环算出 Fibonacci 数？

这里从下标 0 开始，`F(0) = 0`、`F(1) = 1`，从第 2 项起，`F(i) = F(i - 1) + F(i - 2)`。想求的是第 `n` 项的值，便无需保存整张数列表。

每轮开始时，`older`、`newer` 分别存 `F(i-2)` 和 `F(i-1)`。**先把两项相加存到临时变量，再移动旧值**。若先用 `newer` 覆盖 `older` 再求和，两个加数就会变成同一项。初始一对是 `(0, 1)`，三轮更新后依次为 `(1, 1)`、`(1, 2)`、`(2, 3)`。

```c
#include <inttypes.h>
#include <stdio.h>

static uint32_t fibonacci_iterative(uint32_t n)
{
    if (n <= 1) return n;
    uint32_t older = 0, newer = 1;
    for (uint32_t i = 2; i <= n; ++i) {
        uint32_t next = older + newer;
        older = newer;
        newer = next;
    }
    return newer;
}

int main(void)
{
    uint32_t n = 10;
    if (n > 47) return 1;
    printf("F(%" PRIu32 ") = %" PRIu32 "\n", n, fibonacci_iterative(n));
    return 0;
}
```

录屏在 Visual Studio 实际运行 `n = 10`，显示 55、退出码 0。录屏的打印标签读变量 `number`，函数却直接传字面量 `10`；只改变量会让标签与计算脱节。复习版在两处均使用 `n`。循环从 2 到 10，共做九次加法。讲师提醒注意 `++i` 与 `i++`；在本例的 `for` 更新位置，表达式的值被丢弃，两种写法都令下标增加 1。

整理者在调用处补了 32 位边界。`F(47) = 2971215073` 仍可用 `uint32_t` 表示，`F(48) = 4807526976` 已经超出。这个迭代算法所做的加法次数随 `n` 线性增加，只用固定数量的变量。录屏把递归版留作练习，没有测量两版的耗时或内存。
