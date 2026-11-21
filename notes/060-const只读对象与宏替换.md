# 060 · 单价用 const、税率用 #define，两者怎样参与计算？

## 用单价、税额和总价区分两种写法

课堂约定一件商品的单价为 100，练习税率为 13%。税额是 `100 × 0.13 = 13`，含税总价是 `100 + 13 = 113`。下面保留课堂的计算，`printf` 是整理者补全的作业。

```c
#include <stdio.h>
#define TAX_RATE 0.13

int main(void)
{
    const double price = 100.0;
    const double tax = price * TAX_RATE;
    const double total = price + tax;

    printf("price=%.2f, tax=%.2f, total=%.2f\n", price, tax, total);
    return 0;
}
```

整理者实测输出 `price=100.00, tax=13.00, total=113.00`。`%.2f` 控制显示的小数位数，变量存储的值不随显示格式改变。课堂只写到 `tax`、`total` 和格式提示，没有打印结果。

**需要一个有类型、初始化后只读的对象时，用 `const`。** `const double price = 100.0;` 在定义时给出初值。后面再写 `price = 123.0;`，Visual Studio 编辑器在赋值左侧标出 `E0137：表达式必须是可修改的左值`。应删除误写的赋值。如果单价确实需要更新，声明时用 `double price`。税率也可以写成 `const double tax_rate = 0.13;`，类型 `double` 要写全。[C 的赋值规则](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=119)

**需要在预处理阶段按名字替换表达式时，用 `#define`。** 在 `#define TAX_RATE 0.13` 中，宏名后用空格接替换内容。不要在这里写等号或句末分号，否则符号也会被带进表达式。整理者查看预处理输出，税额一行已变成 `const double tax = price * 0.13;`。程序不会创建名为 `TAX_RATE` 的变量。大写用来提醒读者这是宏，C 语法也允许小写。把定义放在文件顶部是为了让后文能使用它，宏从定义处起生效，与 `main` 的运行位置无关。[GCC 对象式宏说明](https://gcc.gnu.org/onlinedocs/cpp/Object-like-Macros.html)

要亲自看宏展开，把示例保存为 `price.c`，在 Visual Studio 开发人员命令提示符中进入文件所在目录，运行 `cl /nologo /EP price.c`。在输出中找 `const double tax =` 那一行，检查乘号后是否已经是 `0.13`。[微软预处理选项 /EP](https://learn.microsoft.com/zh-cn/cpp/build/reference/ep-preprocess-to-stdout-without-hash-line-directives?view=msvc-170)

## 在 Visual Studio 复现课堂的 C17 设置

1. 使用 `.c` 源文件。打开 **项目 → 项目属性 → 配置属性 → 常规**，确认顶部的配置和平台是你要更改的那一组。课堂选择的是 `Debug / ARM64`。
2. 找到 **C 语言标准**，选 **ISO C17 (2018) 标准 (`/std:c17`)**，点“确定”。重新打开属性，看到这一行仍为 C17 即可。旁边的 **C++ 语言标准** 控制 C++ 源文件。若当前属性页布局不同，可在 **配置属性 → C/C++ → 语言** 找同名字段。

微软文档核对日期为 2026-11-21。未显式指定 C 标准时，MSVC 文档列出的默认 C 模式是 C89 加微软扩展。`const` 在默认模式下也可用，选择 C17 是为了指定本项目采用的语言标准。录屏下拉列表里的 `/std:clatest` 表示工具集已实现的后续草案及实验功能。[微软 C17 项目设置](https://learn.microsoft.com/zh-cn/cpp/overview/install-c17-support?view=msvc-180#configure-c11-or-c17-mode-in-visual-studio) · [微软 C 标准选项](https://learn.microsoft.com/zh-cn/cpp/build/reference/std-specify-language-standard-version?view=msvc-180#c-standards-support)
