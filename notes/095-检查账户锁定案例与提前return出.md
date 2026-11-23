# 095 · 账户锁定后，怎样用 return 停止后续流程？

讲师用变量 `wrong_attempts` 模拟已经输错密码的次数。锁定条件写成 `>= 3`，这样第 3 次就触发，超过 3 次也仍然满足。录屏里通过修改初值切换情形，没有读取密码或累计次数。

```c
#include <stdbool.h>
#include <stdint.h>
#include <stdio.h>

int main(void) {
    uint32_t wrong_attempts = 3;
    bool is_locked = wrong_attempts >= 3;

    if (is_locked) {
        puts("账户被锁定!");
        return 0;
    }

    puts("继续往后执行");
    return 0;
}
```

`is_locked` 已经是布尔值，可以直接写 `if (is_locked)`。值为假时，程序跳过整个花括号，执行下面的 `puts`。值为真时，程序先打印锁定提示，再执行块内的 `return 0;`。锁定路径已经从 `main` 返回，所以后续语句直接写在 `if` 下方即可，无需另写 `else`。[C 的 if 语句](https://learn.microsoft.com/zh-cn/cpp/c-language/if-statement-c?view=msvc-170)

**怎样验证流程真的停了？** 讲师固定次数为 3，保留块外的输出，先运行上面的逻辑，再移除块内的 `return 0;` 运行，最后恢复它再运行。三次都打印锁定提示，只有中间一次还打印了“继续往后执行”。两处 `return 0;` 都使程序正常结束，单看退出码 0 无法区分走了哪条路径。若把同样的判断写在普通函数中，`return` 只会返回调用处，调用者会从那里继续。[C 的 return 语句](https://learn.microsoft.com/zh-cn/cpp/c-language/return-statement-c?view=msvc-170)

复习时只改 `wrong_attempts` 的初值，每次重新编译运行。上例简化了录屏的输出标点。整理者实测，设为 2 只打印“继续往后执行”，设为 3 或 4 都只打印“账户被锁定!”。
