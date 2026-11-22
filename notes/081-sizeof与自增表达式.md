# 081 · sizeof 遇到前缀自增，变量会变吗？

`sizeof ++e` 按 `sizeof(++e)` 分组，整个 `++e` 都是 `sizeof` 的操作数。这里 `e` 和 `++e` 的类型都是 `int`，编译器凭类型就能确定字节数。`++e` 不求值，因此 `e` 保持初值。

```c
#include <stdio.h>

int main(void) {
    int e = 5;
    size_t bytes = sizeof ++e;

    printf("bytes = %zu\n", bytes);
    printf("e = %d\n", e);
    return 0;
}
```

课堂在 Visual Studio 的 Debug ARM64 配置下运行这段代码，输出两行。

```text
bytes = 4
e = 5
```

第一行用 `%zu` 打印 `sizeof` 返回的无符号整数类型 `size_t`。换一个 C 实现，`bytes` 仍等于 `sizeof(int)`，具体字节数可以不同。第二行用 `%d` 打印 `int e`，可以直接核对它有没有从 `5` 变成 `6`。[C11 草案 §6.5.3.4](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=108) · [printf 格式 §7.21.6.1](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=329)

只想查变量 `e` 的大小，写 `sizeof e` 或 `sizeof(e)` 即可。前一种写法能省括号，因为 `sizeof` 是运算符，可以直接接变量名。要查类型的大小，类型名必须放进括号，如 `sizeof(int)`。[C11 草案 §6.5.3](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=106)

**标准补充**　若操作数的类型是变长数组，`sizeof` 会对它求值，数组的总字节数可以到运行时才确定。[C11 草案 §6.5.3.4 第 2 段](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=108)
