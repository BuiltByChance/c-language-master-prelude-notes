# 032 · unsigned int：为什么 −1 会打印成 4294967295

## 同样占 4 字节，能表示哪些数？

在 Microsoft Learn 的“数据类型范围”表中，对照“字节”和“值的范围”两列。课堂所用 MSVC 的 `int` 和 `unsigned int` 都占 4 字节。前者能表示 −2147483648 到 2147483647，后者能表示 0 到 4294967295。C 中 `signed`、`signed int` 与 `int` 同义，`unsigned` 与 `unsigned int` 同义。换编译环境时，从 `<limits.h>` 读取 `UINT_MAX` 就能得到该环境中 `unsigned int` 的上限。[微软数据类型范围](https://learn.microsoft.com/zh-cn/cpp/cpp/data-type-ranges?view=msvc-170) · [C 标准委员会草案 §6.7.3.1](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf#page=116)

## 先确定变量存的值，再选打印格式

在源文件开头写 `#include <stdio.h>`，把下面两行放进 `main`。在 Visual Studio 选择 **调试 → 开始执行（不调试）**，课堂的 Debug x64 配置打印 `4294967173`。

```c
unsigned int positiveNum = -123;
printf("%u\n", positiveNum);
```

转换发生在初始化时。把整数转成 `unsigned int`，结果按数学上的 `UINT_MAX + 1` 取模。在课堂环境中，模数是 2³²，因此 −123 存成 4294967296 − 123 = **4294967173**。把初值改为 −1 会得到 **4294967295**，改为 0 或 123 就保留原值。相同的初值和类型会得到相同的结果。[C 整数转换规则 §6.3.1.3](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf#page=60)

`printf` 的 `%u` 按 `unsigned int` 输出十进制，`%d` 要求 `int`。课堂把上例改用 `%d`，屏幕出现了 `-123`，但变量仍保存 `4294967173`。这个值超出 `int` 上限，错配 `%d` 的行为未定义，不能靠换格式找回原输入。[C 格式化输出规则 §7.23.6.1](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf#page=345)

## 阳光值只允许 0 到 100000，在哪一步检查？

若把输入 `-1` 先存进 `unsigned int`，它就和输入 `UINT_MAX` 得到同一个值，此时 `value < 0` 永远为假。要把负数归零、过大的数封顶，就要在仍能分辨正负时比较。

十万是课堂设定的上限。讲师写了注释和赋值，没有运行条件分支。一行注释写成 `>10000`，上限变量和口述都是 `100000`。下面是整理者补写的函数，沿用课堂的 MSVC 范围，假定输入已经读成 `long`。

```c
unsigned int sunlight_from_input(long input)
{
    const long limit = 100000;
    if (input < 0) {
        return 0;
    }
    if (input > limit) {
        input = limit;
    }
    return (unsigned int)input;
}
```

用 `-9`、`0`、`80`、`100000`、`100001` 调用，期望依次得到 `0`、`0`、`80`、`100000`、`100000`。

## 循环下标必须无符号吗？

C 允许整数类型作下标。从数组首元素访问长度为 `n` 的数组时，合法下标满足 `0 ≤ i < n`。`int` 能表示所需范围时可以使用它。对于实际声明的数组 `a`，可以用 `size_t n = sizeof a / sizeof a[0];` 取得元素个数。用 `size_t` 作下标便于和 `n` 比较，仍须检查 `i < n`。若用 `-1` 表示“未找到”，应在转成无符号下标前处理这个值。[数组下标 §6.5.3.2](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf#page=88) · [`size_t` 的定义 §7.21](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf#page=327)
