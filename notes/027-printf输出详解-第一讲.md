# 027 · printf 输出详解（第一讲）：整数的进制、对齐与补零

## 一个 `%d` 对应哪个值

`printf` 从左到右把格式串里的转换说明与后面的实参配对。`%d` 按有符号十进制打印一个 `int`。把课堂教材的算式简写为 `printf("%d - %d = %d\n", ten, 2, ten - two);`，当两个 `int` 变量 `ten` 为 10、`two` 为 2 时，输出就是 `10 - 2 = 8`。三个位置依次取变量、整数常量和表达式的结果。

- **确认变量有值。** `int dogs, cats = 94;` 只给 `cats` 写了初值。函数内这样定义的普通局部变量 `dogs` 应先赋值再打印。
- **核对类型。** `22` 是 `int` 常量，`22.0` 和 `2.2E1` 都是 `double` 常量，数值相等也要按各自的类型选择转换说明。
- **再查个数。** Visual Studio 的 **错误列表（Error List）** 出现 C4473 时，从左到右检查每个需要值的转换说明，补齐对应实参或删掉多余的转换说明。课堂中少传实参的程序带警告运行了，但 C 对这种调用的结果没有规定。[微软 C4473 说明](https://learn.microsoft.com/en-us/cpp/error-messages/compiler-warnings/c4473?view=msvc-170) · [C 标准委员会草案 §7.21.6.1](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=327)

## 让同一个 100 显示成不同进制

课堂用 `int number = 100;` 演示。下面是可独立运行的复习示例（整理者改写），给 `%d` 传 `int`，给 `%o`、`%x`、`%X` 传 `unsigned int`。[C 标准委员会草案 §7.21.6.1](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=330)

```c
#include <stdio.h>

int main(void) {
    int number = 100;
    unsigned int u = (unsigned int)number;
    printf("%d %o %x %X\n", number, u, u, u);  // 100 144 64 64
    printf("%#o %#x %#X\n", u, u, u);          // 0144 0x64 0X64
    return 0;
}
```

只改转换说明，变量的值保持 100。`%x` 用 `a` 到 `f`，`%X` 用 `A` 到 `F`，所以十六进制数 `64` 看不出字母大小写。需要让读者一眼认出进制时，可加 `#`。此时八进制首位为 `0`，非零十六进制数带 `0x` 或 `0X`。课堂控制台验证了两个十六进制前缀，`0144` 展示在教材页。[微软格式说明](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/format-specification-syntax-printf-and-wprintf-functions?view=msvc-170)

## 控制对齐、符号和补位

读 `%-10d` 这类格式时，从 `%` 后依次看标志、最小字段宽度和转换字符。下表的 `·` 代表一个实际空格。

| 格式 | 对 `100` 的输出 | 怎样读 |
| --- | --- | --- |
| `%10d` | `·······100` | 默认右对齐，左侧补 7 个空格 |
| `%-10d` | `100·······` | `-` 让数字靠左，右侧补 7 个空格 |
| `%010d` | `0000000100` | `0` 让左侧用零补到总宽 10 |
| `%+d` | `+100` | 显式打印符号，传 `-100` 就显示 `-100` |
| `% d` | `·100` | 非负数前留一个空格，负数在同一位置显示负号 |

**自测（整理者补充）。** 写 `printf("|%-10d|\n", 100);`，两根竖线会框出 **10 个字符**。竖线是照原样打印的普通字符，不计入字段宽度。若数字本身超过指定宽度，`printf` 会完整输出。[微软字段宽度说明](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/format-specification-syntax-printf-and-wprintf-functions?view=msvc-170#width)
