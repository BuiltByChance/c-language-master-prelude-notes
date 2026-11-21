# 059 · 怎样判断 char 能否存 200，什么时候用 unsigned char 或 uint8_t？

## 要存 200，先查哪一项？

课堂打开的是微软《字符值的范围》。在文档描述的 MSVC 环境里，一个字节有 8 位，`signed char` 的范围是 −128～127，`unsigned char` 是 0～255。普通 `char` 是第三种类型，MSVC 默认让它与 `signed char` 同范围，编译选项 `/J` 可让它与 `unsigned char` 同范围。录屏展示了选项说明，没有修改项目设置。[微软《字符值的范围》](https://learn.microsoft.com/zh-cn/cpp/c-language/range-of-char-values?view=msvc-170) · [C11 对三种字符类型的定义](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=59)

换平台时，`sizeof(char)` 总是 1，只能告诉你它占一个 C 字节。查 `<limits.h>` 的 `CHAR_BIT` 才知道每字节有几位。要存负数，看 `CHAR_MIN` 是否小于或等于目标值。要存 200，看 `CHAR_MAX` 是否至少为 200。对只取 0～255 的灰度值，`unsigned char` 保证能容纳全部取值。[C11 整型范围](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=46) · [sizeof 的单位](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=108)

课堂用一盒 8 支彩笔举例。记录每盒还剩多少支只需 0～8。若把一批盒子的计数存成数组，用 `unsigned char` 每盒占 1 字节，改用 `unsigned int` 每盒占 `sizeof(unsigned int)` 字节。

**整理者复测。** 把下面的程序作为 `.c` 文件，用项目实际采用的编译选项运行。

```c
#include <limits.h>
#include <stdio.h>

int main(void)
{
    unsigned char gray = 200;
    printf("CHAR_BIT=%d, char=%d..%u\n",
           CHAR_BIT, (int)CHAR_MIN, (unsigned)CHAR_MAX);
    printf("signed char=%d..%d, unsigned char=0..%u\n",
           (int)SCHAR_MIN, (int)SCHAR_MAX, (unsigned)UCHAR_MAX);
    printf("gray=%u\n", (unsigned)gray);
    return 0;
}
```

Windows x64、TinyCC 0.9.27 默认配置实测为 `CHAR_BIT=8, char=-128..127`、`signed char=-128..127, unsigned char=0..255`、`gray=200`。在这份配置下，把灰度值存入普通 `char` 就无法保留全部 256 个等级。

## 文本、原始字节和协议字段怎样选？

| 要表达的数据 | 写法与依据 |
| --- | --- |
| 给 C 字符串接口传文本 | 用 `char[]`，用 `'\0'` 标记字符串末尾。若日志消息是文本，先看接口要求的字符类型。 |
| 逐字节检查或复制对象的内存表示 | 用 `unsigned char[]`。C 明确允许用 `memcpy` 把对象的表示复制到这样的数组中。原始数据可能含 `0`，应另存字节数，按长度处理。 |
| 协议明确规定为 8 位的无符号字段 | 在 `<stdint.h>` 提供 `uint8_t` 的实现上用它，值域恰好为 0～255，没有填充位。这个保证只覆盖该字段。多字节字段还要约定字节序，直接发送 C 结构体还可能把成员间的填充一起发出去。 |

本次复测所用的头文件写着 `typedef unsigned char uint8_t;`，在这个实现里，两种写法是同一类型，都能运算。在别的平台，`unsigned char` 始终存在，`uint8_t` 则取决于实现能否提供恰好 8 位的无符号整数类型。[C11 对象的字节表示](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=62) · [精确宽度整数](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=308) · [结构体填充](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=133)
