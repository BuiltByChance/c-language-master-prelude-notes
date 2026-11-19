# 029 · scanf 已经过时了吗？分清 C 标准、C4996 和 C6031

`scanf` 仍是 `<stdio.h>` 中的标准 C 函数，作业指定它时可以照题写。讲师说后续课程会沿用微软运行库提供的 `scanf_s`。换到其他工具链，要先确认运行库是否提供 `scanf_s`，因为它属于 C 标准的可选接口。[C 标准委员会草案 §7.23.6.4](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf#page=357) · [附录 K 的适用条件](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf#page=637)

## 在 Visual Studio 运行要求用 `scanf` 的作业

1. 打开 **视图 → 输出**，选择 **生成**。课堂把 `scanf_s("%d", &score)` 改成 `scanf("%d", &score)` 后，这里显示 `error C4996`。它来自微软运行库对 `scanf` 的弃用标记，项目的编译设置可以把这条警告提升为错误。[微软 C4996 说明](https://learn.microsoft.com/en-us/cpp/error-messages/compiler-warnings/compiler-warning-level-3-c4996?view=msvc-170)
2. 若要保留 `scanf`，在调用它的 `.c` 文件中，把 `#define _CRT_SECURE_NO_WARNINGS` 放在所有 `#include` 之前。编译器处理头文件时就需要看到这个定义。宏名前写 `//` 只会得到注释。这个设置抑制该源文件里这类微软运行库弃用提示，读入规则和返回值照旧。[微软运行库兼容性说明](https://learn.microsoft.com/en-us/cpp/c-runtime-library/compatibility?view=msvc-170)
3. 保存并重新生成，确认 **1 个成功、0 个失败**。启动程序，键入 `88` 并按回车。课堂控制台下一行打印了 `88`，退出代码为 `0`。

## 构建成功后，为什么还有波浪线

讲师悬停时看到的是 **C6031**，含义是忽略了 `scanf` 的返回值。`scanf("%d", &score)` 只读一项，返回 `1` 才表示这项已写入。处理这个结果，就能区分有效的零分和读取失败。[微软 C6031 说明](https://learn.microsoft.com/en-us/cpp/code-quality/c6031?view=msvc-170)

下面的复习示例保留课堂的宏和 `scanf`，由整理者补上失败分支。输入 `88` 会打印 `88`，输入 `abc` 会显示 `No integer was read.`。

```c
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>

int main(void) {
    int score = 0;
    printf("Enter an integer score:\n");
    if (scanf("%d", &score) != 1) {
        printf("No integer was read.\n");
        return 1;
    }
    printf("%d\n", score);
    return 0;
}
```
