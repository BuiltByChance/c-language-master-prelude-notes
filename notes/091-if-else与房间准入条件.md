# 091 · 四个房间的准入规则：怎样组合 &&、|| 和括号？

程序用同一名玩家的属性依次检查四个房间。同一人可能获准进入多个房间，因此每个房间各写一组 `if / else`。如果把四个房间串成 `else if`，一旦 A 放行，B、C、D 就都不会再检查。

| 房间 | 进入条件 | 课堂实跑：VIP、15 枚金币、无特殊道具 |
| --- | --- | --- |
| A | 是 VIP | 可进 |
| B | 至少 10 枚金币，或有特殊道具 | 可进，15 枚已够 |
| C | 是 VIP，且至少 20 枚金币 | 不可进，还差 5 枚 |
| D | 至少 5 枚金币，且具备 VIP 身份或特殊道具中的任意一项 | 可进，VIP 身份和金币数均达标 |

## 从文字规则写到代码

把下面的程序保存为 `.c` 文件，改动三个初值后重新编译运行，每次应看到四行结果。整理者沿用课堂条件，将录屏中的拼写 `has_speacil_item` 统一为 `has_special_item`，并简写提示文字。这里用 `puts` 输出固定字符串，它会自动在末尾换行。要把变量的值填进提示文字，可用 `printf` 的格式说明符。[Microsoft Learn · puts](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/puts-putws?view=msvc-170)

```c
#include <stdio.h>
#include <stdbool.h>
#include <inttypes.h>

int main(void) {
    bool is_vip = true;
    uint32_t coins = 15;
    bool has_special_item = false;

    if (is_vip) {
        puts("A 可进");
    } else {
        puts("A 不可进：需要 VIP");
    }

    if (coins >= 10 || has_special_item) {
        puts("B 可进");
    } else {
        puts("B 不可进：需要至少 10 枚金币或特殊道具");
    }

    if (is_vip && coins >= 20) {
        puts("C 可进");
    } else {
        puts("C 不可进：需要 VIP 且至少 20 枚金币");
    }

    if ((is_vip || has_special_item) && coins >= 5) {
        puts("D 可进");
    } else {
        puts("D 不可进：需要 VIP 或特殊道具，并且至少有 5 枚金币");
    }
    return 0;
}
```

## 怎么知道 D 的括号有没有写对？

在 B，道具可以替代 10 枚金币。在 D，道具只能替代 VIP 身份，5 枚金币仍须另查。若写成 `is_vip || has_special_item && coins >= 5`，因为 `&&` 的优先级高于 `||`，C 会将 `has_special_item && coins >= 5` 分成一组，金币门槛就只约束持道具的路径。课堂的 15 枚金币在两种写法里都会放行，单凭那次输出发现不了漏判。[Microsoft Learn · C 运算符优先级](https://learn.microsoft.com/zh-cn/cpp/c-language/precedence-and-order-of-evaluation?view=msvc-170)

整理者补测时固定为“VIP、无道具”，只改金币数。4 枚时正确写法拒绝进入 D，去掉内层括号却会放行。增至恰好 5 枚时，正确写法也放行，验证了“不少于”包含等于。再换成“非 VIP、有道具、4 枚金币”，B 放行，D 仍拒绝。
