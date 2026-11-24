# 111 · 猜数游戏怎样提示大小，遇到无效输入又该从哪里重试？

一局只出一道题。进入循环前用 `srand((unsigned)time(NULL))` 设一次种子，再用 `rand() % 100 + 1` 取一次答案。余数是 0～99，加 1 后是 1～100。若把取答案的语句放进循环，上一次的“太大”或“太小”就不能用来缩小下一次的范围。[Microsoft Learn《srand》](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/srand?view=msvc-170) · [《rand》](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/rand?view=msvc-170)

## 先猜一次，再判断要不要继续

录屏把读取猜测和比较大小放进 `do` 块。`guess < secret_num` 提示太小，`guess > secret_num` 提示太大。相等时两个分支都跳过，尾部 `while (guess != secret_num);` 为假，才执行循环外的祝贺语。如果把第二个分支简写成 `else` 并打印“太大”，猜中时也会打印这句。

第一局依次输入 `20`（太大）、`15`（太小）、`18`（太大），答案已经缩到 16 或 17。输入 `16` 后显示答案 16，进程以 0 退出。

## 同样是输入字母，为什么会有三种结果？

| 录屏中的读法 | 控制台现象 | 原因 |
| --- | --- | --- |
| 直接 `scanf_s("%d", &guess)`，忽略返回值 | 输入 `qq` 后不断交替打印“太大”和输入提示 | `%d` 在第一个 `q` 处匹配失败，既没给 `guess` 赋新值，也没取走 `q`。下一轮还会读到它。 |
| 转换失败就执行 `while (getchar() != '\n')`，把报错和 `continue` 放在里面 | 8 个字母报 8 次错，随后还多报一次“太大” | 内层循环每读一个字母就报错。读到换行退出内层后又执行了大小比较。把比较移入成功分支，只解决了多报大小。 |
| 先 `fgets(buffer, sizeof buffer, stdin)`，再用 `sscanf_s` 从字符串读数 | 一行字母只报一次错，随后可以再猜，现场最终猜中 88 | 这次输入的字母和换行都已读进 `buffer`。转换失败后，下一次 `fgets` 从下一行开始。 |

**末版还有一个首轮错误。** `continue` 跳到最近一层循环的判断处。若第一次输入 `q`，末版会直接执行 `guess != secret_num`，此时 `guess` 还没有值。保留原结构时，可把尾条件改为 `while (status != 1 || guess != secret_num);`。左边为真时，C 会跳过右边的比较。读入前还要检查 `fgets` 是否返回 `NULL`，否则输入结束后仍会尝试解析缓冲区。[Microsoft Learn《continue》](https://learn.microsoft.com/zh-cn/cpp/c-language/continue-statement-c?view=msvc-170) · [《fgets》](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/fgets-fgetws?view=msvc-170)

录屏用 `%d` 读 `uint32_t guess`。`%d` 要与 `int *` 配对。保留 `uint32_t` 时用 `"%" SCNu32`，返回的字段数用 `int status` 接收。即使类型正确，单个 `%d` 返回 1 也只说明读到了一个整数前缀，`16abc` 仍能读出 16。录屏输入 `231` 得到“太大”，说明游戏也没有限制猜测范围。[C11 委员会草案 §7.8.1](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=235) · [Microsoft Learn《sscanf_s》](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/sscanf-s-sscanf-s-l-swscanf-s-swscanf-s-l?view=msvc-170)

## 整理者复习程序

在 Visual Studio 的 C 控制台项目中，把代码放进含 `main` 的 `.c` 文件，按 `Ctrl+F5`，每行猜一个数。这个版本只在读到 1～100 的完整整数后才比较大小。尾条件只看已初始化的 `won`，因此第一轮输错也能安全重试。

```c
#include <ctype.h>
#include <stdbool.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

int main(void)
{
    srand((unsigned)time(NULL));
    const int secret_num = rand() % 100 + 1;
    bool won = false;
    char line[50];

    do {
        puts("请猜一个 1 到 100 的整数：");
        if (fgets(line, sizeof line, stdin) == NULL) {
            puts("输入结束，本局结束。");
            return ferror(stdin) ? 1 : 0;
        }
        if (strcspn(line, "\n") == sizeof line - 1) {
            int ch;
            do {
                ch = getchar();
            } while (ch != '\n' && ch != EOF);
            puts("这一行过长，请重新输入。");
            continue;
        }

        char *rest;
        long guess = strtol(line, &rest, 10);
        while (isspace((unsigned char)*rest)) {
            ++rest;
        }
        if (*rest != '\0' || guess < 1 || guess > 100) {
            puts("输入无效，请输入 1 到 100 的整数。");
            continue;
        }

        if (guess < secret_num) {
            puts("太小了。");
        } else if (guess > secret_num) {
            puts("太大了。");
        } else {
            won = true;
        }
    } while (!won);

    printf("猜对了！答案是 %d。\n", secret_num);
    return 0;
}
```

`line[50]` 为换行和字符串结尾各留一格，允许一行正文最多 48 个字节。如果读满 49 个字节仍未遇到换行，就丢弃本行剩余部分，整行只报一次错。`strtol` 按十进制读取，`rest` 指向第一个没读进去的字符。跳过空白后若还剩 `a`，`16abc` 就会被拒绝。没有数字时返回 0，数值溢出时返回 `LONG_MIN` 或 `LONG_MAX`，都落在本题范围外。[Microsoft Learn《strtol》](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strtol-wcstol-strtol-l-wcstol-l?view=msvc-170)

**复测方法。** 临时把 `secret_num` 固定为 `16`。先输入 `q`、空行、`16abc`、`231`，应分别只报一次错并再次询问。再输入 `20`、`15`、`16`，应依次看到“太大”“太小”“猜对”，之后不再询问。
