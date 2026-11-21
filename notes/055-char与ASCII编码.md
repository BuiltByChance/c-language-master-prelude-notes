# 055 · 为什么在 ASCII 环境中用 %c 打印 98 会得到 b？

ASCII（American Standard Code for Information Interchange）为字符分配整数编号。在使用 ASCII 兼容编码的环境中，小写 `b` 的编号是 98。把整数 98 交给 `printf`，`%c` 写出对应字符 `b`，`%d` 写出十进制数字 `98`。其他 C 实现可以采用不同的执行字符集，要表示字母时直接写 `'b'` 更可靠。[ASCII 码表](https://man7.org/linux/man-pages/man7/ascii.7.html) · [C11 字符输出规则](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=332)

## 为什么变量是 char，悬停却显示 int？

在 Visual Studio 的控制台项目中，把下面的程序放进 `.c` 文件，按 Ctrl+F5 运行。第一行输出沿用课堂写法，第二行是整理者补充的对照。

```c
#include <stdio.h>

int main(void)
{
    char mych = 'A';
    printf("mych: %c\n", mych);
    printf("code: %d\n", (int)mych);
    return 0;
}
```

在课堂的编码环境中，两行应分别显示 `mych: A` 和 `code: 65`。普通字符常量 `'A'` 在 C 中的类型是 `int`，所以鼠标指向单引号里的常量会看到 `(int)65`。左边的 `mych` 则按声明存为 `char`，`sizeof mych` 等于 1 个 C 字节。单引号 `'A'` 给出字符值，双引号 `"A"` 给出带结尾空字符的字符串。[C11 字符常量](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=87) · [字符串字面量](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=88) · [sizeof](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=108)

片末有一次构建失败，错误列表提示 `identifier "chat" is undefined`。提示点名的是声明开头的类型拼写，把 `chat` 改回 `char`，重新运行就显示了 `mych: A`。

## 怎样查码表，判断屏幕上会出现什么？

ASCII 用 7 位表示 **128 个码值，编号从 0 到 127**。在码表的 `Dec`（十进制）列找 `97`，同一行的 `Hex` 是 `61`，`Char` 是小写 `a`。把它写成 8 位二进制就是 `01100001`，三种进制表示同一个编号。[ASCII 码表及控制码定义](https://www.rfc-editor.org/rfc/rfc20.html) · [按 Dec 列查表](https://man7.org/linux/man-pages/man7/ascii.7.html)

| 十进制码值 | 怎样解读 |
| --- | --- |
| 0–31、127 | 0–31 是控制码，127 是 DEL（删除）。13 是 CR（`\r`），使光标回到本行开头。10 是 LF（`\n` 在课堂环境中的值），表示换行。 |
| 32–126 | 可打印字符。32 是空格，44 是逗号，65 是 `A`，97 是 `a`，98 是 `b`。 |

课堂把 `int mych` 设为 13 后，`printf("mych: %c\n", mych)` 仍然打印标签。`%c` 写出回车，末尾的 `\n` 再换行，因此标签后看不到字形。要验证输出了什么，可以另用 `%d` 打印同一个值。

在 ASCII 表中，`A`–`Z` 为 65–90，`a`–`z` 为 97–122。两段起点相差 32，因此只有先确认输入是 ASCII 小写字母，才能用减 32 得到对应的大写字母。汉字超出了这张表。Unicode 保留 ASCII 的前 128 个码点，UTF-8 对这部分沿用相同的单字节值，汉字等其他字符则使用多个字节。[ASCII 字母表](https://man7.org/linux/man-pages/man7/ascii.7.html) · [UTF-8 编码规则](https://www.rfc-editor.org/rfc/rfc3629.html#section-3)
