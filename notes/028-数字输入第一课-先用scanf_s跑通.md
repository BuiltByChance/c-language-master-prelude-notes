# 028 · 数字输入第一课：用 scanf_s 读入整数分数

`scanf_s("%d", &score)` 从标准输入读一个十进制整数。`&score` 是 `int` 变量的地址，函数把读到的数写到这里。`printf("%d\n", score)` 则取 `score` 当前的值，把它显示出来。[微软 scanf_s 文档](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/scanf-s-scanf-s-l-wscanf-s-wscanf-s-l?view=msvc-170)

## 跑通一次输入，再试一次失败

课堂把 `score` 初始化为 0，读完就打印。输入 `88` 后屏幕显示 `88`，输入字母后却显示 `0`。后一次没有读到整数，打印的是初值。下面的复习程序增加了返回值检查（整理者补充）。

```c
#include <stdio.h>

int main(void) {
    int score = 0;
    printf("Enter an integer score:\n");
    if (scanf_s("%d", &score) != 1) {
        printf("No integer was read.\n");
        return 1;
    }
    printf("%d\n", score);
    return 0;
}
```

把程序写入 Visual Studio 控制台项目的 `.c` 文件，选 **调试 → 开始执行（不调试）**（Ctrl+F5）。每次运行时，在提示后键入表中的一个值并按回车。

| 键入 | `scanf_s` 返回值 | 这段程序的输出 |
| --- | ---: | --- |
| `88` | `1` | `88` |
| `0` | `1` | `0`，成功读入零分 |
| `abc` | `0` | `No integer was read.` |

返回值是成功写入的项目数。这里仅读一个整数，所以成功时返回 `1`。若一开始就遇到输入结束，则返回 `EOF`。`%d` 只需匹配开头的整数部分，例如输入 `88.5` 会读入 `88`，留下 `.5`。只检查返回值还不能证明整行都是整数。[微软 scanf_s 文档](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/scanf-s-scanf-s-l-wscanf-s-wscanf-s-l?view=msvc-170) · [C 标准委员会草案 §7.21.6.2](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=335)

## 提示语和输入格式里的 `\n`

- **输出提示。** `printf` 的提示末尾有 `\n`，光标就移到下一行。删掉它，输入会紧跟在冒号后。课堂两种排版都成功读入了 `88`。提示文字写到标准输出，`scanf_s` 读取的是标准输入。
- **读取整数。** 输入格式写 `"%d"`。若写 `"%d\n"`，末尾的空白指令会继续读空格、制表符和换行，直到遇到下一个非空白字符或输入结束。在交互控制台里，输完 `88` 按一次回车后，程序可能还在等待。讲师运行前删掉了这个 `\n`。上述等待原因是按规范补充的。[C 标准委员会草案 §7.21.6.2](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=335)

## 改成 `scanf` 后，怎样看 Visual Studio 的诊断

课堂删掉 `_s` 后，当前项目构建失败。弹窗询问是否运行上次成功生成的程序时，选 **否**，再看**错误列表**。选“是”会运行旧程序，无法验证刚改的代码。[微软构建与运行选项](https://learn.microsoft.com/en-us/visualstudio/ide/configure-build-run-options?view=visualstudio)

- **C4996** 是微软运行库对 `scanf` 的弃用诊断，项目设置可以把它提升为错误。讲师把调用改回 `scanf_s` 后停在编辑器。换工具链时，需要确认目标 C 运行库是否提供 `scanf_s`，C 标准将它列为可选接口。[微软 C4996 说明](https://learn.microsoft.com/en-us/cpp/error-messages/compiler-warnings/compiler-warning-level-3-c4996?view=msvc-170) · [C 标准委员会草案附录 K](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=601)
- **C6031** 提示忽略了函数返回值。上面示例中的 `!= 1` 就是对这次输入是否成功的检查。[微软 C6031 说明](https://learn.microsoft.com/en-us/cpp/code-quality/c6031?view=msvc-170)
