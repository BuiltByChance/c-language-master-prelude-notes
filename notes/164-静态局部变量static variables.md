# 164 · 怎样让函数记住调用次数，又把计数器留在函数内部？

在函数体内声明 `static int count = 0;`。名字 `count` 只在其声明所在的块中可见，对象却具有静态存储期，整个程序运行期间只有这一份。C 在程序启动前完成它的初始化。每次进入函数只执行 `count++` 和打印，函数返回时保留递增后的值。

以下是按课堂程序缩写的复习示例：

```c
#include <stdio.h>

void visit(void)
{
    static int count = 0;
    ++count;
    printf("visit count = %d\n", count);
}

int main(void)
{
    visit();
    visit();
    visit();
    return 0;
}
```

录屏用显式 `= 0` 运行，计数依次为 **1、2、3**；删掉 `= 0` 再运行，仍打印 **1、2、3**，退出码为 0。把声明换为普通的 `int count = 0;`，每次调用都会新建并初始化一个对象，三次就都打印 **1**。两次实际输出相同，因为无显式初值的静态存储期整数会自动初始化为零。[WG14 C11 草案 §6.2.4](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n1570.pdf#page=56) · [§6.7.9 第 10 段](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n1570.pdf#page=158)

**判断 `static` 的含义先看声明位置**。此处的块内 `count` 没有链接，其他函数不能靠另一个同名声明访问它。把 `static int count;` 写到函数外，它仍长期存在，但名字具有内部链接，可以由同一翻译单元中声明之后的多个函数使用。[WG14 C11 草案 §6.2.2](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n1570.pdf#page=54)
