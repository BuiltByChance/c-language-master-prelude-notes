# 142 · 为什么“中”和一个笑脸的字节数不同，怎样让 Windows 控制台正确显示它们？

Unicode 码点给字符编号。UTF-8 和 UTF-16 将同一个编号写成不同的代码单元。UTF-8 的一个单元是 **8 位，即 1 字节**，UTF-16 的一个单元是 16 位。下面是录屏中 Windows ARM64 程序的实际读数，均不含字符串结尾的零。[Unicode 联盟的 UTF 问答](https://www.unicode.org/faq/utf_bom.html)

| 字符 | Unicode 码点 | UTF-8 字节（十六进制） | Windows 宽字符串中的 UTF-16 单元（十六进制） |
| --- | --- | --- | --- |
| 中 | U+4E2D | `E4 B8 AD`，3 字节 | `4E2D`，1 个单元 |
| 😀 | U+1F600 | `F0 9F 98 80`，4 字节 | `D83D DE00`，2 个单元 |

`wchar_t wide[] = L"😀";` 是宽字符数组。在录屏的 MSVC 环境里，`sizeof(wchar_t)` 是 2，笑脸需要一对代理项。`sizeof wide / sizeof wide[0] - 1` 得到 2，扣掉的是结尾的一个宽空字符。换一个 C 实现需要重新确认 `wchar_t` 的大小及编码。[Microsoft 的字符类型说明](https://learn.microsoft.com/en-us/cpp/cpp/char-wchar-t-char16-t-char32-t?view=msvc-170)

**在 Windows 上复现片末的 UTF-8 输出**

1. 将 `.c` 文件保存为 UTF-8。若文件打开时已经乱码，先按原编码重新打开，确认文字正确，再另存为 UTF-8。重新打开只改变编辑器怎样解读现有字节。讲师在记事本的“文件 → 另存为”展示了编码列表。若旧文件约定为 GBK，读取时仍需使用 GBK。[WHATWG 的 GBK 规范](https://encoding.spec.whatwg.org/#gbk)
2. 在 Visual Studio 中右击项目，打开“属性 → 配置属性 → C/C++ → 命令行 → 其他选项”，添加 `/utf-8` 并应用。它同时设置源字符集和执行字符集，保证此处的普通字符串字面量按 UTF-8 编进程序。录屏展示了如何输入这个选项，点确定前已清空。[Microsoft 的 /utf-8 文档](https://learn.microsoft.com/zh-cn/cpp/build/reference/utf-8-set-source-and-executable-character-sets-to-utf-8?view=msvc-170)
3. 在控制台项目中运行以下程序。`SetConsoleOutputCP(CP_UTF8)` 设置当前控制台的输出代码页。若字节正确却显示为缺字方框，还应检查终端字体是否含对应字形。录屏还展示了系统区域设置中已勾选的 UTF-8 测试选项，程序没有改动它。[Windows 的 SetConsoleOutputCP 文档](https://learn.microsoft.com/en-us/windows/console/setconsoleoutputcp)

```c
#include <stdio.h>
#include <windows.h>

int main(void)
{
    SetConsoleOutputCP(CP_UTF8);
    printf("中文：中\n");
    printf("Emoji: 😀\n");
    return 0;
}
```

讲师先尝试宽输出，控制台出现问号。源码一度把宽字符串交给需要单个宽字符的 `%lc`，Visual Studio 提示类型不匹配。宽字符串应使用 `%ls`，单个 `%lc` 也容纳不了上表中的两个 UTF-16 单元。节目没有验证那套宽输出写法成功。末次换成上面的普通 `printf` 后，控制台实际显示“中”和笑脸，退出码为 0。
