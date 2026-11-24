# 143 · 为什么同一个 1.5 在控制台有时显示为 1,50？

`double price = 1.5;` 保存的是同一个数。`printf("%.2f", price)` 指定显示两位小数，数字格式的区域设置 `LC_NUMERIC` 决定中间用点还是逗号。程序启动时的区域设置是 `"C"`。录屏先打印 `1.50`，请求德语（德国）`"de-DE"` 后打印 `1,50`，恢复 `"C"` 后又打印 `1.50`。

下面是整理者按录屏改写的例子。系统若不支持给定的区域名，`setlocale` 返回 `NULL`，原设置继续生效，因此应先查返回值，再解读结果。

```c
#include <locale.h>
#include <stdio.h>

int main(void)
{
    double price = 1.5;
    printf("startup = %.2f\n", price);
    if (setlocale(LC_NUMERIC, "de-DE") != NULL) {
        printf("de-DE = %.2f\n", price);
    } else {
        puts("de-DE unavailable");
    }
    setlocale(LC_NUMERIC, "C");
    printf("restored C = %.2f\n", price);
    return 0;
}
```

| 第二个参数 | 调用的效果 |
| --- | --- |
| `"C"` | 明确切回标准 C 区域，`1.5` 用点分隔。 |
| `""` | 请求运行环境的默认区域。录屏第二次改成空字符串后也看到 `1.50`，程序的输出标签却还写着 `de-DE`，仅凭一个点号无法认定当前区域是 `"C"`。 |
| `NULL` | 只查询当前设置。可用 `setlocale(LC_NUMERIC, NULL)` 读取当前数字区域的名称，用 `localeconv()->decimal_point` 读取小数分隔符。 |

课程最后口述把空字符串说成默认的 `"C"`。两者调用意图不同，在小数点一样的系统上输出可以碰巧相同。[Microsoft 的 setlocale 文档](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/setlocale-wsetlocale?view=msvc-170)
