# 040 · 用 inttypes.h 打印定宽整数：为什么同一个 uint64_t 有两种格式

打印 `uint64_t value` 的十进制值，可以写 `printf("%" PRIu64 "\n", value);`。`<inttypes.h>` 提供格式宏。`PRI` 表示输出，`u` 选无符号十进制，`64` 对应变量类型的位宽。换成有符号的 `int32_t`，就用 `PRId32`。这些宏从 C99 起成为标准，输入用的宏另以 `SCN` 开头。[C99 整数格式宏](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=210)

`%` 留在引号内，宏名放在引号外。课堂里 `PRIu64` 展开为字符串 `"llu"`，因此 `"%" PRIu64 "\n"` 先变成 `"%" "llu" "\n"`，编译时再拼成 `"%llu\n"`。引号外的空格不会打印。在 glibc 2.39 的 64 位字长分支，同一行代码会拼出 `"%lu\n"`。直接写死 `%llu` 要求实参是 `unsigned long long`，仅知道它占 64 位还不够。[C 字符串拼接规则](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=74) · [glibc 的长度前缀](https://github.com/bminor/glibc/blob/glibc-2.39/stdlib/inttypes.h#L43-L49) · [PRIu64 定义](https://github.com/bminor/glibc/blob/glibc-2.39/stdlib/inttypes.h#L102-L105)

## 用四个最大值核对格式和实参

下面的复习程序保留课堂运行过的三行，并修正最后的 8 位示例。录屏中标为 `int8_t` 的两行分别用了 `%hhd` 和 `PRId8`，传入的却都是 `myInt32`。检查这种错误时，看逗号后的变量及其声明。引号内的 `int8_t:` 只是打印出来的标签。

```c
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void)
{
    int32_t  myInt32  = INT32_MAX;
    uint32_t myUInt32 = UINT32_MAX;
    uint64_t myUInt64 = UINT64_MAX;
    int8_t   myInt8   = INT8_MAX;

    printf("int32_t: %"  PRId32 "\n", myInt32);
    printf("uint32_t: %" PRIu32 "\n", myUInt32);
    printf("uint64_t: %" PRIu64 "\n", myUInt64);
    printf("int8_t: %"   PRId8  "\n", myInt8);
    return 0;
}
```

把代码放进 Visual Studio 的 C 控制台项目，选择 `Debug` 和 `x64`，按 **Ctrl+F5**。若要核对本机采用哪种格式，在编辑器按住 **Ctrl** 单击宏名，读取 `#define` 右侧的字符串。[Visual Studio 转到定义](https://learn.microsoft.com/en-us/visualstudio/ide/go-to-and-peek-definition?view=vs-2022)

| 格式宏 | 课堂头文件里的定义 | 程序应打印的十进制值 |
| --- | --- | ---: |
| `PRId32` | `"d"` | 2147483647 |
| `PRIu32` | `"u"` | 4294967295 |
| `PRIu64` | `"llu"` | 18446744073709551615 |
| `PRId8` | `"hhd"` | 127 |

前三个数都出现在课堂控制台。最后的 `127` 是整理者改用 `myInt8` 后的实测结果。字幕把 `hhd` 识别为“SSD”。`hh` 是长度修饰符，配上 `d` 表示按 `signed char` 的值输出十进制。[微软 printf 格式说明](https://learn.microsoft.com/en-us/cpp/c-runtime-library/format-specification-syntax-printf-and-wprintf-functions?view=msvc-170#argument-size-specification)
