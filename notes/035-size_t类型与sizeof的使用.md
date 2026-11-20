# 035 · sizeof 与 size_t：sizeof(short) 得到的 2 该怎么读

在课堂的 Windows x64 项目里，`sizeof(short)` 的值是 2，表示一个 `short` 对象占 **2 字节**。`sizeof` 是求字节数的运算符，结果的类型叫 `size_t`，是一种无符号整数类型。要知道存放这个计数本身要几个字节，写 `sizeof(size_t)`。微软文档给出的 `size_t` 在 x86 下是 4 字节的 `unsigned int`，在 x64 下是 8 字节的 `unsigned __int64`。[C 标准委员会的 sizeof 定义](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf#page=95) · [微软《最大数组大小》](https://learn.microsoft.com/zh-cn/cpp/c-language/largest-array-size?view=msvc-170)

用 `printf("%zu\n", sizeof(short));` 打印这个数。小写 `u` 表示按无符号十进制输出，前面的 `z` 指定这个参数的类型是 `size_t`。`%d` 要求 `int`，单写 `%u` 要求 `unsigned int`，两者都不能跨平台匹配 `size_t`。[C 格式化输出规则](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf#page=345)

## 在 Visual Studio 复现并比较两个目标

把下面的整理者复习示例放进 C 控制台项目的 `.c` 文件。前四行输出把课堂的八个类型按有符号、无符号配对。最后一行新增了一个比较：`bytes` 里存的是 `short` 的大小，`sizeof bytes` 量的是变量 `bytes` 自己的大小。

```c
#include <stddef.h>
#include <stdio.h>

int main(void)
{
    size_t bytes = sizeof(short);
    printf("short / unsigned short: %zu / %zu\n", bytes, sizeof(unsigned short));
    printf("int / unsigned int: %zu / %zu\n", sizeof(int), sizeof(unsigned int));
    printf("long / unsigned long: %zu / %zu\n", sizeof(long), sizeof(unsigned long));
    printf("long long / unsigned long long: %zu / %zu\n",
           sizeof(long long), sizeof(unsigned long long));
    printf("size_t: %zu\n", sizeof bytes);
    return 0;
}
```

在 Visual Studio 顶部工具栏选择 `Debug` 和 `x64`，按 **Ctrl+F5** 编译并运行。记下控制台结果，关闭控制台，把平台切为 `x86` 后再按一次 **Ctrl+F5**。可以在“输出”窗口的“生成”记录中确认本次编译的目标。下表单位为字节，斜杠两侧依次对应有符号和无符号类型。

| Windows 目标 | short | int | long | long long | size_t |
| --- | --- | --- | --- | --- | --- |
| x64 | 2 / 2 | 4 / 4 | 4 / 4 | 8 / 8 | 8 |
| x86 | 2 / 2 | 4 / 4 | 4 / 4 | 8 / 8 | 4 |

课堂实测了 x64 行的前四列。第二行输出标签误拼为 `unsigned shart`，源码传给 `sizeof` 的是 `unsigned short`。讲师只打开了 x86 选项。整理者另用 Windows TinyCC 对两个目标分别编译、运行了上面的完整示例，得到表中全部数值。

这些固定整数类型的大小在编译时就已确定，把同一个可执行文件复制到另一台能运行它的电脑，输出仍然相同。若换目标重新编译，先看目标采用的数据模型：Windows x64 的 LLP64 保留 4 字节的 `long`，64 位 macOS 的 LP64 则使用 8 字节的 `long`。仅凭“64 位”无法推断 `sizeof(long)`。[微软 Windows 数据模型](https://learn.microsoft.com/en-us/windows/win32/winprog64/abstract-data-models) · [Apple 64 位数据模型](https://developer.apple.com/library/archive/documentation/Darwin/Conceptual/64bitPorting/transition/transition.html)
