# 183 · 怎样用 size_t 遍历数组，并通过指针改写每一项？

课堂用 `int32_t` 保存十个整数，用 `size_t` 保存元素个数和下标。两种类型的选择依据不同：前者要求元素恰为 32 位有符号整数，后者与 `sizeof` 的无符号结果类型一致。只有在 `arrays` 是完整数组的地方，才能用 `sizeof arrays / sizeof arrays[0]` 得到项数。即使 `i` 是无符号数，仍须检查 `i < count`；等于长度就已超过最大下标。[WG14 C11 草案 §6.5.3.4、§7.19](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n1570.pdf)

```c
#include <inttypes.h>
#include <stdio.h>

int main(void)
{
    int32_t arrays[] = {10, 20, 30, 40, 50, 60, 70, 80, 90, 100};
    int32_t *ptr = arrays;
    size_t count = sizeof arrays / sizeof arrays[0];

    for (size_t i = 0; i < count; ++i)
        *(ptr + i) += 5;

    for (size_t i = 0; i < count; ++i)
        printf("%" PRId32 "%c", arrays[i], i + 1 == count ? '\n' : ' ');
    return 0;
}
```

`ptr + i` 得到第 `i` 项的地址；`*(ptr + i) += 5` 等价于 `arrays[i] += 5`。地址计算没有改动 `ptr`，赋值改动的是原数组。课堂分别打印修改前后的十项，首尾由 10、100 变为 15、105，完整结果为 `15 25 35 45 55 65 75 85 95 105`。上例保留写入步骤，并通过数组名重新读取以验证修改。

录屏里按住 Ctrl 点击类型名，打开 VS 的 `vcruntime.h`，可见 `_WIN64` 分支把 `size_t` 定义为 `unsigned __int64`，把指针差类型 `ptrdiff_t` 定义为有符号的 `__int64`。这些宽度属于该实现。打印时按实参选择：元素 `int32_t` 用 `PRId32`，长度 `size_t` 用 `%zu`，同一数组内的指针差用 `%td`。[WG14 C11 草案 §7.8.1、§7.21.6.1](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n1570.pdf)

循环末尾单独使用 `++i` 或 `i++` 都把这个整数下标增加 1。前缀表达式产生新值，后缀表达式产生旧值；该位置丢弃表达式的值，课堂没有测量性能差异。
