# 033 · short 与 unsigned short：存屏幕坐标和 RGB565 颜色

## 两个字节够不够？

| 课堂 MSVC 类型 | 每个值占用 | 能存的整数 |
| --- | --- | --- |
| `short`（也写作 `short int`、`signed short int`） | 2 字节 | −32768 到 32767 |
| `unsigned short`（也写作 `unsigned short int`） | 2 字节 | 0 到 65535 |
| `int` | 4 字节 | −2147483648 到 2147483647 |

先列出数据的最小值和最大值，再决定能否缩小类型。例如要存 100000，上表两种短整数都不够。如果整批数据都在范围内，用 `short` 数组替换同样元素个数的 `int` 数组，课堂环境中的存储量能减半。换平台时，可查 `<limits.h>` 中的 `SHRT_MIN`、`SHRT_MAX` 和 `USHRT_MAX`。[微软数据类型范围](https://learn.microsoft.com/zh-cn/cpp/cpp/data-type-ranges?view=msvc-170) · [整数上下限](https://learn.microsoft.com/zh-cn/cpp/c-language/cpp-integer-limits?view=msvc-170)

## 打印 800 × 600 画面内的一个点

像素从 0 编号，800 × 600 画面内的横坐标为 0～799，纵坐标为 0～599，因此宽、高和点 `(400, 300)` 的两个分量都能存进 `unsigned short`。先确认坐标相对于哪个原点。比如 Windows 扩展桌面的副屏位于主屏左边时，副屏上的点可以有负横坐标，这时就要选能容纳该范围的有符号类型。[微软虚拟屏幕坐标说明](https://learn.microsoft.com/en-us/windows/win32/gdi/the-virtual-screen)

在 Visual Studio 的 `.c` 源文件中运行下面的课堂示例。四个无符号数都是预设值，运行时无需调整显示器或点击鼠标。

```c
#include <stdio.h>

int main(void)
{
    short myshort = 100;
    unsigned short screenWidth = 800, screenHeight = 600;
    unsigned short x = 400, y = 300;

    printf("myshort = %hd\n", myshort);
    printf("The screen resolution is : %hu X %hu\n", screenWidth, screenHeight);
    printf("Current position is : (%hu, %hu)\n", x, y);
    return 0;
}
```

选择 **调试 → 开始执行（不调试）**，控制台三行应依次显示 `myshort = 100`、`The screen resolution is : 800 X 600`、`Current position is : (400, 300)`。格式中的 `h` 表示短整数，`d` 和 `u` 分别指定有符号、无符号十进制。普通 `short` 传给 `printf` 时会提升为 `int`，所以 `%d` 也能正确输出 `myshort`。写成 `%s` 会让 `printf` 期待字符串指针。[微软 printf 格式说明](https://learn.microsoft.com/en-us/cpp/c-runtime-library/format-specification-syntax-printf-and-wprintf-functions?view=msvc-170)

## 为什么 0xF800 是红色，查网页却出现绿色？

RGB565 从高位到低位依次用 **5 位红、6 位绿、5 位蓝**。纯红取各通道值 `(31, 0, 0)`，拼成 `11111 | 000000 | 00000`。把这 16 位每四位一组重排，就是 `1111 1000 0000 0000`，即 `0xF800`。[微软 RGB565 位布局与掩码](https://learn.microsoft.com/en-us/windows/win32/directshow/working-with-16-bit-rgb)

课堂用二进制常量 `0b1111100000000000` 赋值。复习时可把上例 `main` 中的声明和打印语句换成下面两行，保留 `return 0;`。这里改用十六进制，较早的标准 C 模式也能编译。`%04hX` 把无符号短整数打印成至少四位的大写十六进制数，不足四位时在左侧补零。[C23 二进制整数常量 §6.4.4.2](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf#page=70) · [printf 长度与宽度字段](https://learn.microsoft.com/en-us/cpp/c-runtime-library/format-specification-syntax-printf-and-wprintf-functions?view=msvc-170)

```c
unsigned short color = 0xF800;
printf("RED : 0x%04hX\n", color);
```

控制台输出 `RED : 0xF800`。课堂打开的绿色网页按六位色码 `#00F800` 解读，把数字分为 `00 | F8 | 00`，于是红、绿、蓝分别为 0、248、0。查色时要连同格式一起查。课堂程序验证了数值，没有绘制色块。[W3C 网页十六进制颜色记法](https://www.w3.org/TR/css-color-4/#hex-notation)
