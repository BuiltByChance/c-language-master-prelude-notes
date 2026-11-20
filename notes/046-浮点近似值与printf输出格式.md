# 046 · 同一个 0.1F，怎样读懂 %.20f、%E 和 %A 的输出

## 让五行输出读取同一个值

在 Visual Studio 控制台项目中，右击「源文件」→「添加」→「新建项」，命名为 `display_float.c`。录屏用的是 `Debug | ARM64`。把下面的代码写入文件，按 **Ctrl+F5** 生成并运行。

```c
#include <stdio.h>

int main(void)
{
    float value = 0.1F;
    printf("%.6f\n", value);
    printf("%.20f\n", value);
    printf("%e\n", value);
    printf("%E\n", value);
    printf("%A\n", value);
    return 0;
}
```

| 格式 | 课堂最后一次运行的输出 | 怎样读 |
| --- | --- | --- |
| `%.6f` | `0.100000` | 舍入到小数点后 6 位，这个近似值与十分之一的差别在此处看不出来。 |
| `%.20f` | `0.10000000149011611938` | 显示小数点后 20 位，能看出存入的值略大于十分之一。 |
| `%e`、`%E` | `1.000000e-01`、`1.000000E-01` | 十进制科学记数法。`E-01` 表示乘以 10⁻¹。两行的有效数字相同，只有指数字母的大小写不同。 |
| `%A` | `0X1.99999A0000000P-4` | 读作十六进制数 `1.99999A` 乘以 2⁻⁴。`P` 后的指数用十进制写，末尾的十六进制小数零可以省略。 |

要核对二进制有效数字，可以用 `%a` 或 `%A`。每个十六进制数字对应四个二进制位，例如输出中的 `A` 展开为 `1010`。[微软 printf 格式说明](https://learn.microsoft.com/en-us/cpp/c-runtime-library/format-specification-syntax-printf-and-wprintf-functions?view=msvc-170#type-field-characters)

## 十分之一在哪一步变成近似值？

1. **写常量时就要选一个能表示的值。** 十分之一写成二进制是 `0.0001100110011…₂`，其中的 `0011` 会循环。课堂使用的 binary32 正规数有 23 位显式小数，加上隐含的首位 1，共 24 位有效数字。整理者核算，离十分之一最近的 binary32 值恰好是 `0.100000001490116119384765625`。换用 binary64 可以更接近十分之一，仍然写不尽这个循环；而 0.5 等于二进制的 `0.1₂`，两种格式都能精确保存。[Oracle 的 IEEE 单精度与双精度格式说明](https://docs.oracle.com/cd/E60778_01/html/E60763/z4000ac019878.html)
2. **看后缀判断常量类型，看声明判断变量类型。** `0.1` 是 `double` 常量，`0.1f` 和 `0.1F` 是 `float` 常量，`0.1L` 和 `0.1l` 是 `long double` 常量。因此，录屏中的 `double value2 = 0.1L;` 用 `long double` 常量初始化一个 `double` 变量。五行输出用的都是 `value`。[微软 C 浮点常量说明](https://learn.microsoft.com/en-us/cpp/c-language/c-floating-point-constants?view=msvc-170)
3. **传给 `printf` 时，提升保留了已有的近似值。** `value` 作为可变参数传入时从 `float` 提升为 `double`，原来的变量仍是 `float`。五次调用之间没有运算或重新赋值；从六位改成二十位，只会让同一个数的更多小数位显示出来。[微软 printf 参数转换说明](https://learn.microsoft.com/en-us/cpp/c-runtime-library/format-specification-syntax-printf-and-wprintf-functions?view=msvc-170#type)
