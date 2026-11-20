# 034 · long 与 long long：为什么同是 x64，long 的大小还会不同

## 先看编译目标，再选整数类型

CPU 架构规定机器能执行哪些指令。x64（也称 x86-64）涵盖 AMD64 和 Intel 64。一个 C 程序里 `long` 和指针各占几个字节，还取决于目标平台采用的**数据模型**，也就是编译器生成这个程序时遵守的类型大小约定。[微软 x64 架构说明](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/x64-architecture)

| 编译目标及数据模型 | `int` | `long` | `long long` | `void *` 指针 |
| --- | ---: | ---: | ---: | ---: |
| Windows x86，MSVC（ILP32） | 4 | 4 | 8 | 4 |
| Windows x64，MSVC（LLP64） | 4 | 4 | 8 | 8 |
| Linux x86-64，常见的 LP64 目标 | 4 | 8 | 8 | 8 |

表中单位均为字节。课堂板书把“x64 上 `long` 占 8 字节”当成通则。对照后两行就能看出差别：两者的指针都是 8 字节，Windows 的 `long` 仍为 4 字节。课堂打开的微软类型表同时适用于 32 位和 64 位 MSVC。[微软数据类型范围](https://learn.microsoft.com/en-us/cpp/cpp/data-type-ranges?view=msvc-170) · [Windows 数据模型](https://learn.microsoft.com/en-us/windows/win32/winprog64/abstract-data-models) · [GCC 的 x86 目标选项](https://gcc.gnu.org/onlinedocs/gcc/x86-Options.html)

`long`、`long int`、`signed long int` 是同一种类型。C 保证 `long` 至少 32 位、`long long` 至少 64 位，大小关系是 `sizeof(short) ≤ sizeof(int) ≤ sizeof(long) ≤ sizeof(long long)`，相邻两项可以等宽。整理者举例：若要在上述三个目标上都保存有符号计数 30 亿，可写 `long long count = 3000000000LL;`。这个值超过 Windows 的 `LONG_MAX`（2147483647）。打印 `long long` 用 `%lld`，打印 `long` 用 `%ld`。[C 标准委员会草案的整数范围](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf#page=36) · [微软 printf 格式说明](https://learn.microsoft.com/en-us/cpp/c-runtime-library/format-specification-syntax-printf-and-wprintf-functions?view=msvc-170)

## 在自己的 C 项目里量一次

课堂只查表和板书。下面是整理者补充的复习程序，`sizeof` 返回字节数，`%zu` 用来打印这个结果。

```c
#include <stdio.h>

int main(void)
{
    printf("int=%zu long=%zu long long=%zu pointer=%zu\n",
           sizeof(int), sizeof(long), sizeof(long long), sizeof(void *));
    return 0;
}
```

在 Visual Studio 的 **生成 → 配置管理器** 中，将 C 项目的平台分别选为 `Win32` 和 `x64`，每次重新生成并运行。整理者用 Tiny C Compiler 在 Windows 上另行生成、运行了这两个目标，依次得到 `4/4/8/4` 和 `4/4/8/8`。对照输出时看最后一项：这次切换改变了指针宽度，两次的 `long` 都是 4。[微软项目目标平台设置](https://learn.microsoft.com/en-us/visualstudio/ide/how-to-configure-projects-to-target-platforms)

## 安装包、老软件和内存各看什么？

- **选安装包看系统类型。** Windows 的 **设置 → 系统 → 关于 → 系统类型** 会列出操作系统位数和处理器类型。厂商同时提供原生包时，基于 x64 的 64 位系统选 x64，基于 ARM 的 64 位系统选 ARM64，32 位 x86 系统选 x86。Windows 11 只有 64 位版本（2026-11-20 核实）。[微软系统类型说明](https://support.microsoft.com/en-au/windows/experience/compatibility/32-bit-and-64-bit-windows-frequently-asked-questions)
- **判断兼容性看程序本身。** 安装包标签 `x86` 通常指 32 位，而在处理器历史中，x86 家族从 16 位的 8086 起步，Intel386 才引入 32 位通用寄存器。x64 Windows 可通过 WOW64 运行许多 32 位 Windows 程序，但不能直接运行 16 位 Windows 程序。课堂没有测试《红色警戒 2》的具体版本，不能凭发行年代判定故障原因。[Intel 架构手册 §2.1](https://cdrdv2-public.intel.com/782149/253665-sdm-vol-1.pdf#page=35) · [微软 WOW64 说明](https://learn.microsoft.com/en-us/windows/win32/winprog64/running-32-bit-applications)
- **查内存上限要分清物理内存和进程地址空间。** 普通 32 位 Windows XP 的物理内存上限为 4 GB。装满 4 GB 后，设备地址映射可能使系统可用量变少，具体数值随硬件而变。单个 32 位进程默认可用的 2 GB 用户虚拟地址空间是另一项限制。[微软 Windows 内存限制表](https://learn.microsoft.com/en-us/windows/win32/memory/memory-limits-for-windows-releases#physical-memory-limits-windows-xp)
