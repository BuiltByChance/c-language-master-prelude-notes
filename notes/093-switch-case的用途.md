# 093 · 怎样用 switch 按角色选择规则？break 和 default 各管什么？

`switch (role)` 适合按一个整数编号选择规则。每个 `case` 后面写一个整数常量表达式，各个 `case` 的值不能重复。比如 `role` 为 2 时从 `case 2:` 开始，即使 1 和 2 在 `if (role)` 中都算真，`switch` 也能区分它们。任务数是否超过门槛，要在选中角色后再判断。[Microsoft Learn：C 的 switch 语句](https://learn.microsoft.com/zh-cn/cpp/c-language/switch-statement-c?view=msvc-170)

## 一次访问判断怎样走完

先把 `access_allowed` 设为 `false`。讲师的第一版在各分支里按下表决定是否改为 `true`，退出 `switch` 后再根据这个布尔值统一输出。

| 入口 | 何时允许访问 |
| --- | --- |
| `case 1` 经理 | 无条件允许 |
| `case 2` 普通员工 | 完成任务数 `> 5`，即至少 6 项 |
| `case 3` 实习生 | 完成任务数 `> 10`，即至少 11 项，且经理已批准 |
| `default` 其他编号 | 一律拒绝 |

**命中了角色，仍可能被拒绝。** 普通员工只做了 5 项，程序已经进入 `case 2`，内层 `if` 条件为假，权限保持初值。把 `break;` 写在内层 `if` 的右花括号之后，任务数达标与否都会退出 `switch`。省掉它，程序会接着执行下一分支的语句，不会重新比较角色编号。只有角色编号没有匹配任何 `case` 时，才会从 `default` 开始执行。[Microsoft Learn：C 的 break 语句](https://learn.microsoft.com/zh-cn/cpp/c-language/break-statement-c?view=msvc-170)

## 规则需要单独修改时，把判断移入函数

让每个角色的函数返回“是否允许”，`switch` 只负责选择函数并接收返回值。以后调整员工的任务门槛，只改 `check_access_for_emp`。新增角色则要加一个 `case`。录屏在 `main` 前声明函数、在其后写函数体。下面的复习版把定义移到 `main` 前，可以整段保存为 `.c` 文件编译运行。

```c
#include <stdbool.h>
#include <stdint.h>
#include <stdio.h>

bool check_access_for_manager(void) {
    return true;
}

bool check_access_for_emp(uint8_t completed_tasks) {
    return completed_tasks > 5;
}

bool check_access_for_intern(uint8_t completed_tasks, bool is_manager_approved) {
    return completed_tasks > 10 && is_manager_approved;
}

int main(void) {
    uint8_t role = 2;
    uint8_t completed_tasks = 6;
    bool is_manager_approved = false;
    bool access_allowed = false;

    switch (role) {
    case 1:
        access_allowed = check_access_for_manager();
        break;
    case 2:
        access_allowed = check_access_for_emp(completed_tasks);
        break;
    case 3:
        access_allowed = check_access_for_intern(completed_tasks, is_manager_approved);
        break;
    default:
        access_allowed = false;
        break;
    }

    if (access_allowed) {
        puts("允许访问");
    } else {
        puts("拒绝访问");
    }
    return 0;
}
```

**怎样验证两个条件缺一不可？** 讲师现场运行的是拆函数前的版本。普通员工完成 6 项、未经经理批准，控制台显示“允许访问”。整理者另行编译了上面的函数版，把 `role` 改为 3 后，以 10 项且获批为起点，输出“拒绝访问”。只把任务数增加到 11 项，输出“允许访问”。保持 11 项并撤掉批准，输出又变为“拒绝访问”。此时把角色改为 4，即使重新批准，也会走 `default` 输出“拒绝访问”。
