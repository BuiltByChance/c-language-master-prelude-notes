# 113 · 既要忽略越界数字，又要用 -1 退出，先检查哪一个？

输入 `-1` 就结束，其他负数和大于 100 的数跳过，只有 0～100（含两端）需要判断奇偶。`-1` 同时满足“退出”和“小于零”，所以先检查退出值。如果先做范围筛选，`continue` 会把 `-1` 当普通负数略过，程序将继续等输入。

## 把三条路径写成一个循环

在 Visual Studio 的 C 控制台项目中，把下列整理者复习程序放进含 `main` 的 `.c` 文件，按 `Ctrl+F5`。`number` 用有符号类型，才能区分 `-1` 和其他整数。代码沿用讲师的判断顺序，额外检查读取结果。每个数字应能由 `int32_t` 表示。

```c
#include <stdio.h>
#include <inttypes.h>

int main(void)
{
    int32_t number;
    while (1) {
        puts("请输入 0 到 100 的整数，输入 -1 结束：");
        if (scanf_s("%" SCNd32, &number) != 1) {
            fputs("未读到整数。\n", stderr);
            return 1;
        }
        if (number == -1) {
            break;
        }
        if (number < 0 || number > 100) {
            continue;
        }
        if (number % 2 == 0) {
            printf("%" PRId32 " 是偶数。\n", number);
        } else {
            printf("%" PRId32 " 是奇数。\n", number);
        }
    }
    puts("程序结束。");
    return 0;
}
```

**验证顺序。** 输入 `-2 0 3 100 101 -1 4`。`-2` 和 `101` 后面直接是下一次输入提示。`0`、`3`、`100` 分别显示偶、奇、偶。`-1` 后显示“程序结束”，末尾的 `4` 不再处理。这组边界值是整理者补测。

**区分等待与刷屏。** 录屏里，空格加回车后程序安静等待，原因是 `%d` 会略过空白，继续等数字。在格式串前面加空格也没有改变这个结果。输入字母时，整数转换失败，字母留在输入流中。原程序忽略返回值，反复打印提示。复习程序只在返回 `1` 时才使用 `number`，否则报错结束。[Microsoft Learn《scanf 格式字段》](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/format-specification-fields-scanf-and-wscanf-functions?view=msvc-170) · [《scanf_s 返回值》](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/scanf-s-scanf-s-l-wscanf-s-wscanf-s-l?view=msvc-170)
