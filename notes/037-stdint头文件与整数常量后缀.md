# 037 · 用 stdint.h 声明整数：查类型、写最大值、验证字节数

## 在头文件里读懂一个类型

在 C 源文件中写下 `#include <stdint.h>`。在 Visual Studio 里按住 **Ctrl** 单击 `stdint.h`，查找 `int32_t`。课堂打开的微软头文件中写着 `typedef int int32_t;`：最右边的名字是别名，左边的 `int` 是这套编译器实际使用的类型。回到自己的 `.c` 标签页写程序。[微软类型对照表](https://learn.microsoft.com/en-us/cpp/c-runtime-library/standard-types?view=msvc-170#fixed-width-integral-types-stdinth)

| 要求 | 选择 | 课堂 MSVC 示例 |
| --- | --- | --- |
| 字段必须恰好 16 位 | 有符号用 `int16_t`，无符号用 `uint16_t`。实现有相应的精确宽度类型时才提供这些名称。 | `short`、`unsigned short` |
| 至少 16 位，希望少占空间 | `int_least16_t`，选能满足宽度要求的最小存储类型。 | `short` |
| 至少 16 位，优先考虑运算速度 | `int_fast16_t`，由实现选择通常运算最快的合适类型，可以宽于 16 位。 | `int` |

同样的命名规则适用于 8、32、64 位。若只需给 0 到 100 计数，普通 `int` 已经足够。[C99 整数类型定义](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=268)

## 用六个最大值验证：字节数和值各看哪一行

下面的复习程序保留课堂六个初值，把六行大小输出合成三对，最后一行是整理者补充的值检查。

```c
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void)
{
    int16_t  s16 = 32767;
    uint16_t u16 = 65535;
    int32_t  s32 = 2147483647;
    uint32_t u32 = 4294967295U;
    int64_t  s64 = 9223372036854775807LL;
    uint64_t u64 = 18446744073709551615ULL;

    printf("16-bit: %zu / %zu bytes\n", sizeof s16, sizeof u16);
    printf("32-bit: %zu / %zu bytes\n", sizeof s32, sizeof u32);
    printf("64-bit: %zu / %zu bytes\n", sizeof s64, sizeof u64);
    printf("u64 = %" PRIu64 "\n", u64);
    return 0;
}
```

在 Visual Studio 的 C 控制台项目中选 `Debug | x64`，按 **Ctrl+F5**。前三行应依次读到 **2 / 2、4 / 4、8 / 8 字节**，每对依次是有符号和无符号变量。第四行应读到 `18446744073709551615`。哪怕把 `u64` 的初值改为 `1`，它仍占 8 字节，只有第四行会跟着变。

- **读数字末尾的字母。** 在这些十进制常量中，`U` 表示依次从 `unsigned int`、`unsigned long`、`unsigned long long` 中取第一个装得下的类型。`LL`、`ULL` 在本例分别指定 `long long`、`unsigned long long`。后缀决定数字本身的类型，等号左侧的声明决定变量能装多大的值。若要表达“这个类型的最大值”，可直接写 `uint64_t u64 = UINT64_MAX;`，省去手抄 20 位数字。[C99 整数常量选型表](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=68)
- **按实参类型选输出格式。** `sizeof` 返回 `size_t`，用 `%zu`。要打印变量值，课堂 MSVC 的 `int32_t` 是 `int`，应配 `%d`；`%ld` 要求 `long`，即使二者都占 4 字节也不能混用。跨平台打印时，`<inttypes.h>` 的 `PRId32`、`PRIu64` 会给出相应类型的格式，像第四行那样在宏前写 `"%"`。[C99 整数格式宏](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=210)

课堂用 Snipaste 把类型定义贴在编辑器旁。默认按 **F1** 框选、**F3** 置顶。授权核实于 2026-11-20：1.x 个人和商业使用均免费，2.x 个人非商业使用免费，商业使用需付费授权。[Snipaste 官方入口](https://zh.snipaste.com/) · [最终用户许可协议](https://zh.snipaste.com/eula.html)
