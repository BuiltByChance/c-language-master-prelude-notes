# 119 · 平方根有小数，能据此判断一个整数是素数吗？

`sqrt(x)` 求非负数 `x` 的算术平方根，在 `<math.h>` 中声明，参数和返回值均为 `double`。`sqrt(121)` 返回 11。方程 `y² = 121` 还有解 −11，但一次 `sqrt` 调用只返回非负的那个根。[Microsoft Learn《sqrt》](https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/sqrt-sqrtf-sqrtl?view=msvc-170)

在 Visual Studio 的 C 控制台项目中，把以下示例放进 `.c` 文件，按 `Ctrl+F5`。整理者将录屏中两次独立的试验放在一起，输出为 `2.828427` 和 `11.000000`。小数位数由 `printf` 的格式控制。

```c
#include <stdio.h>
#include <math.h>

int main(void)
{
    printf("%.6f\n", sqrt(8.0));
    printf("%.6f\n", sqrt(121.0));
    return 0;
}
```

**判断素数要看整数能否整除。** 大于 1 的整数，如果正因数恰好只有 1 和自身，就是素数。7 块积木只能整分成 1 组 7 块或 7 组 1 块，所以 7 是素数。8 的平方根虽然有小数，积木却能分成 2 组 4 块，所以 8 是合数。2 也是素数，0 和 1 都不在素数范围内。

讲师用积木举例时按数学除法说 `5 ÷ 2 = 2.5`。整理者补充，C 表达式 `5 / 2` 的两个操作数都是整数，结果为 2。判断能否整分应看 `5 % 2`，余数 1 表示有一块分不完。片尾源码中“输入一个数，判断是否为素数”仍是注释，实际运行的程序只计算了平方根。[C11 委员会草案 §6.5.5](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=110)
