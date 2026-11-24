# 106 · 输入 0 就结束，为什么把 break 放在读取和累加之间？

逐个输入非负整数，输入 `0` 结束并打印总和。`number` 保存刚读到的数，`sum` 保存此前已计入的总和，初值为 0。输入次数未知，所以用 `while (true)` 开始每一轮，读到结束标记时用 `break` 跳到循环后的输出语句。

## 先读一个数，再决定要不要加

在 Visual Studio 的 C 控制台项目中，将下面的代码放入含 `main` 的 `.c` 文件，选择“调试”→“开始执行（不调试）”（`Ctrl+F5`）。本例约定每次输入的数及总和都不超过 `UINT32_MAX`（4,294,967,295）。

```c
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>
#include <stdbool.h>

int main(void)
{
    uint32_t sum = 0;
    uint32_t number;

    puts("逐个输入非负整数，输入 0 结束。");
    while (true) {
        puts("请输入一个数：");
        if (scanf_s("%" SCNu32, &number) != 1) {
            fputs("未读到整数。\n", stderr);
            return 1;
        }
        if (number == 0) {
            break;
        }
        sum += number;
    }
    printf("和 = %" PRIu32 "\n", sum);
    return 0;
}
```

- 每输一个数按一次回车。依次输入 `1、2、3、4`，总和依次为 1、3、6、10。再输入 `0`，程序越过本轮的加法，打印 10 并结束。讲师现场跑通了这组输入。整理者补测时，第一个数就输入 `0`，一次加法也没执行，打印 0。
- `break` 退出最近一层包围它的循环或 `switch`。这里的 `if` 决定何时执行 `break`，退出的是 `while`，所以循环后的 `printf` 仍会执行。[Microsoft Learn《break 语句》](https://learn.microsoft.com/zh-cn/cpp/c-language/break-statement-c?view=msvc-170)
- 录屏的读入和打印都用了 `PRIu32`。整理者在示例中按 `<inttypes.h>` 的用途分别使用输入宏 `SCNu32` 和输出宏 `PRIu32`，并补上读取检查。这里只有一个转换项，`scanf_s` 返回 1 才说明新值已写入 `number`。例如输入 `q` 会匹配失败，程序报错结束。讲师在控制台键入后擦除了 `q`。若要让 0 也能作为数据、用 `q` 正常结算，还需单独识别字符命令。[C11 委员会草案 §7.8.1](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=235) · [Microsoft Learn《scanf_s 返回值》](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/scanf-s-scanf-s-l-wscanf-s-wscanf-s-l?view=msvc-170)

## 同一组输入为什么会有不同表现？

| 循环写法 | 第一次判断和读到 0 时会怎样 |
| --- | --- |
| `uint32_t number;` 后直接写 `while (number != 0)` | 函数里的这行代码定义了局部变量，但没有给它初值。第一次判断就读取了它的值。录屏中 Visual Studio 给出静态分析警告 C6001。把第一次读取输入移到第一次比较之前即可消除这个原因。[诊断说明](https://learn.microsoft.com/zh-cn/cpp/code-quality/c6001?view=msvc-170) |
| `number = 0` 后写 `while (number != 0)` | 第一次判断为假，程序连第一个数都没读就输出 0。 |
| `number` 初值非零，循环体先读再加 | 初值只负责进入第一轮，第一次成功读入就覆盖它。讲师分别把初值设为 1 和 1000000，同样输入 `1、2、3、4、0`，都得到 10。此版本执行了一次加 0，到下一次循环判断才退出。 |
| `while (true)` 内先读，再判断是否 `break` | 无需为 `number` 设置占位初值。去掉 `break` 后，即使读到 0，循环仍会提示输入，执行不到末尾的打印。 |

若在 Visual Studio 改好代码后遇到 `LNK1168`，先看错误中“无法打开以进行写入”的文件名。录屏里该文件正是上一次仍在等输入的可执行程序。结束旧进程，再重新生成并运行，输入 `1、2、3、4、0` 后得到 10，才验证了带 `break` 的新版。[Microsoft Learn《LNK1168》](https://learn.microsoft.com/zh-cn/cpp/error-messages/tool-errors/linker-tools-error-lnk1168?view=msvc-170)
