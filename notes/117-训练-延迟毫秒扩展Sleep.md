# 117 · 怎样让倒数每隔约一秒显示一个数？

在循环里先打印当前数字，再暂停当前线程。这样第一项立即出现，下一项要等暂停结束才显示。最后的 `1` 也会停留约一秒，然后才显示“倒数结束”。

| 运行环境 | 头文件与调用 | 参数含义 |
| --- | --- | --- |
| Windows 原生 Visual Studio 工程 | `#include <windows.h>`，`Sleep(1000)` | 大写 `S`，参数以毫秒计。等待结束后还要由系统调度线程运行，不能据此精确计时。[Microsoft Learn《Sleep》](https://learn.microsoft.com/zh-cn/windows/win32/api/synchapi/nf-synchapi-sleep) |
| Linux 的 POSIX 接口 | `#include <unistd.h>`，`sleep(1)` | 小写 `s`，参数以秒计。未被忽略的信号可能提前打断等待。[Linux man-pages《sleep(3)》](https://man7.org/linux/man-pages/man3/sleep.3.html) |

## 用三步倒数验证 Windows 版本

整理者把起点固定为 3，便于验证显示时机。在 Visual Studio 的 C 控制台项目中，把代码放进 `.c` 文件，按 `Ctrl+F5`。观察 `3`、`2`、`1` 是否逐行出现，以及 `1` 和结束语之间是否也有停顿。

```c
#include <stdio.h>
#include <windows.h>

int main(void)
{
    for (int index = 3; index > 0; index--) {
        printf("%d\n", index);
        fflush(stdout);
        Sleep(1000);
    }
    puts("倒数结束");
    return 0;
}
```

录屏先在没有延迟的版本输入 `10`，数字一直输出到 `1`。讲师在同一个 Windows 工程换用 `sleep(1)`，链接器报告 `LNK2019`，无法找到小写的 `sleep`。改回 `Sleep(1000)` 后构建成功，第二次输入 `10`，画面逐步显示到 `3`，随后进程被中止。Linux 写法只作介绍。
