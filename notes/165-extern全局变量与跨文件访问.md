# 165 · 怎样在两个 C 源文件中修改同一个全局整数？

`run_app.c` 定义 `int g_static_var = 10;`，函数名或变量名里出现 `static` 这几个字母不会改变链接属性。此处声明前没有 `static` 关键字，该对象可以通过外部链接被另一个翻译单元使用。[WG14 C11 草案 §6.2.2](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n1570.pdf#page=54)

在 Visual Studio 的“解决方案资源管理器 → 项目 → 源文件”中添加两个 `.c` 文件，让它们参与同一项目的生成，只有 `run_app.c` 定义 `main`。以下为整理者复习版，把双方需要的声明集中在同一项目的 `shared.h`，供两个源文件包含。

**shared.h**

```c
#ifndef SHARED_H
#define SHARED_H

extern int g_static_var;
void try_access_g(void);

#endif
```

**run_app.c**

```c
#include <stdio.h>
#include "shared.h"

int g_static_var = 10;

static void access_g(void)
{
    g_static_var += 5;
    printf("%d\n", g_static_var);
}

int main(void)
{
    printf("%d\n", g_static_var);
    access_g();
    try_access_g();
    return 0;
}
```

**helper.c**

```c
#include <stdio.h>
#include "shared.h"

void try_access_g(void)
{
    g_static_var += 10;
    printf("%d\n", g_static_var);
}
```

**成功标志**：运行后依次得到 **10、15、25**。前两行来自 `run_app.c`，第三行来自 `helper.c`，说明后者在 15 的基础上加了 10。头文件中的 `extern int g_static_var;` 只声明类型和名字；带初值的定义只放在一个源文件中。若把定义加上 `static`，它就不能用来满足另一翻译单元的这个外部引用。

**分辨两种报错**。课中 `helper.c` 起初找不到 `g_static_var`，补上变量的 `extern` 声明才让该处知道要用哪个类型。最终控制台确实显示了 25，但生成输出还有 **C4013**：`run_app.c` 调用 `try_access_g()` 前没见到这个函数的声明。复习版在头文件补齐 `void try_access_g(void);`，两个文件都包含它，使调用与定义的签名可核对。[Microsoft 对 C4013 的说明](https://learn.microsoft.com/en-us/cpp/error-messages/compiler-warnings/compiler-warning-level-3-c4013?view=msvc-170)
