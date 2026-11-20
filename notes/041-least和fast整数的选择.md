# 041 · 存库存和算库存，怎样选 uint_least16_t 和 uint_fast16_t

一项库存最多 50,000 件，数量从 0 起。16 位无符号整数可表示 0～65,535，足以存下每个可能的值。`<stdint.h>` 提供两种适合这个范围的类型，选哪种取决于这份数据怎么用。

| 用途 | 选哪一种 | 选择依据 |
| --- | --- | --- |
| 保存大量库存记录，希望每项少占空间 | `uint_least16_t` | 在至少有 16 个值位的无符号整数类型中，选占用字节数最少的类型。 |
| 取出一项库存参与计算，希望优先考虑运算效率 | `uint_fast16_t` | 由当前 C 实现选择达到相同位宽要求、通常运算最快的无符号类型。具体计算是否提速，要对同一任务计时。 |

两个名字中的 `16` 都是最低位宽。若数据格式要求字段**恰好**是 16 位，应检查目标平台是否提供 `uint16_t`。[C11 委员会草案 §7.20.1.1–3](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=308)

## 在自己的编译环境核对

把下面的课堂程序放进 Visual Studio 的 C 控制台项目，选择本机的目标平台，按 **Ctrl+F5** 运行。`working = stock` 把库存值复制到计算用的变量，两种类型都能表示 50,000，因此复制后数值相同。

```c
#include <stdio.h>
#include <stdint.h>

int main(void)
{
    uint_least16_t stock = 50000;
    uint_fast16_t working = stock;

    printf("least16: value=%u, bytes=%zu\n", (unsigned int)stock, sizeof(stock));
    printf("fast16: value=%u, bytes=%zu\n", (unsigned int)working, sizeof(working));
    return 0;
}
```

读输出时，先看两行的 `value` 是否都是 `50000`，再比较 `bytes`。代码把值转为 `unsigned int` 以匹配 `%u`。`%zu` 打印的是对原变量求 `sizeof` 得到的字节数。

| 同一程序的运行环境 | `sizeof(stock)` | `sizeof(working)` |
| --- | ---: | ---: |
| 讲师实测，Visual Studio，`Debug / ARM64` | 2 | 4 |
| 整理者复测，TinyCC 0.9.27，Windows x64 | 2 | 2 |

想查明字节数的来源，在 Visual Studio 中右键单击 `#include <stdint.h>` 的文件名，选 **转到文档**（F12），查找这两个类型的 `typedef`。课堂的微软头文件分别把它们定义为 `unsigned short` 和 `unsigned int`，与微软的类型表一致。[微软 C 运行库类型表](https://learn.microsoft.com/en-us/cpp/c-runtime-library/standard-types?view=msvc-170#fixed-width-integral-types-stdinth)

若用数组连续保存 100 万个库存数量，元素总占用等于项数乘以单项大小。按课堂读数，选 `least` 占 2,000,000 字节，选 `fast` 占 4,000,000 字节。这是容量推算，课堂实际运行的程序只创建了两个变量。
