# 098 · 红绿灯状态机为什么一次只输出一种灯色？

这个状态机用一个整数保存灯色。每次进入 `switch`，按当前值选中一个分支，先打印当前灯色，再保存下次要用的值。课堂的转换规则是红灯（0）→ 绿灯（1）→ 黄灯（2）→ 红灯（0）。

```c
#include <stdint.h>
#include <stdio.h>

int main(void) {
    uint8_t traffic_light_state = 0;

    switch (traffic_light_state) {
    case 0:
        puts("红灯");
        traffic_light_state = 1;
        break;
    case 1:
        puts("绿灯");
        traffic_light_state = 2;
        break;
    case 2:
        puts("黄灯");
        traffic_light_state = 0;
        break;
    default:
        puts("???");
        break;
    }
    return 0;
}
```

**把状态改成 1，为什么看不到绿灯？** 进入 `case 0` 后，赋值只更新变量，紧接着的 `break` 会退出整个 `switch`。要接着输出绿灯，程序需要保留这个 1，再执行一次 `switch`。若用循环驱动状态机，应在循环外设置初值。每轮都重新初始化为 0 就会反复输出红灯。[Microsoft Learn：C 的 switch 语句](https://learn.microsoft.com/zh-cn/cpp/c-language/switch-statement-c?view=msvc-170)

**怎样同时验证本次输出和下一状态？** 在 `switch` 的右花括号之后、`return 0;` 之前加入下面这行。每次只改初值，重新编译运行。表中是整理者的实测结果。

```c
printf("下一状态 = %u\n", (unsigned)traffic_light_state);
```

| 初值 | 本次输出 | 执行后的状态值 |
| --- | --- | --- |
| 0 | 红灯 | 1 |
| 1 | 绿灯 | 2 |
| 2 | 黄灯 | 0 |
| 3 | `???` | 3 |

`default` 只打印提示，所以未定义的编号 3 会原样保留。课堂用初值 0 启动两次，又将初值改为 1 启动一次，依次看到红灯、红灯、绿灯。最后一次的注释仍写着“初始状态：红灯”，实际生效的是初值 1。复习代码从红灯开始，并把录屏变量名中的 `ligt` 拼写统一为 `light`。

讲师还提到，灯色可用枚举常量命名，单步转换可封装成函数。枚举让读者从名字就能看出目标灯色。要让灯重复切换，仍需反复执行转换逻辑。录屏只写了“枚举”注释。[Microsoft Learn：C 枚举声明](https://learn.microsoft.com/zh-cn/cpp/c-language/c-enumeration-declarations?view=msvc-170)
