# 045 · 定义并输出 float：为什么 48.3f 打印成 48.299999

## 写入变量

初值已知时，写 `float temperature = 36.5f;`，在定义变量时完成初始化。若温度要稍后读取，可以先写 `float temperature;`，等读到数值再赋值。这样的局部变量没有默认初值，第一次取用它的值之前要先赋值。

`36.5f` 末尾的 `f`（也可写 `F`）指定常量类型为 `float`。写 `float temperature = 36.5;` 也合法，右侧的 `36.5` 是 `double`，初始化时转换为 `float`。在课堂的二进制浮点环境中，36.5 可以精确表示，两种写法得到相同的温度值。[微软 C 浮点常量说明](https://learn.microsoft.com/en-us/cpp/c-language/c-floating-point-constants?view=msvc-170)

课堂把温度、湿度和声速分开定义，用逗号把描述同一物体尺寸的长、宽、高写在一行。讲师建议，如果三项经常分别使用，就分行定义。多词变量名遵循项目已有风格，例如下划线式 `speed_of_sound` 或驼峰式 `speedOfSound`。

## 运行并核对输出

把下面的程序写入 Visual Studio 控制台项目的 `.c` 源文件。要复现课堂配置，在工具栏选择 `Debug` 和 `x64`，按 **Ctrl+F5** 构建并运行，对应菜单为 **Debug → Start Without Debugging**。

```c
#include <stdio.h>

int main(void)
{
    float temperature = 36.5f;
    float humidity = 48.3f;
    float speed_of_sound = 343.5e2f;
    float length = 12.34f, width = 23.45f, height = 34.56f;

    printf("Temperature: %f\n", temperature);
    printf("Humidity: %f\n", humidity);
    printf("Speed of Sound: %f\n", speed_of_sound);
    printf("Dimensions (LxWxH): %f x %f x %f\n", length, width, height);
    return 0;
}
```

课堂控制台输出以下四行，进程退出代码为 0。`343.5e2f` 中的 `e2` 表示乘以 10²，因此第三行是 34350。最后一行的三个 `%f` 依次取逗号后的 `length`、`width`、`height`。

```text
Temperature: 36.500000
Humidity: 48.299999
Speed of Sound: 34350.000000
Dimensions (LxWxH): 12.340000 x 23.450001 x 34.560001
```

## 区分存储值和显示位数

讲师现场指出了湿度和尺寸的末位差异，把原因留到下一节。整理者核对得出，`48.3f` 在 binary32 中存成约 `48.299999237`。传给 `printf` 时，这个已有的值自动提升为 `double`。`%f` 默认把它舍入到小数点后六位，因而显示 `48.299999`。[微软 printf 格式与精度说明](https://learn.microsoft.com/en-us/cpp/c-runtime-library/format-specification-syntax-printf-and-wprintf-functions?view=msvc-170#precision)

**整理者补充实验。** 在 `return 0;` 前加上 `printf("%.2f | %.9f\n", length, length);`。同一个 `length` 分别显示为 `12.34 | 12.340000153`。如果需求只是保留两位小数，用 `%.2f` 即可。原输出末尾的零只能说明显示到了第六位，不能据此判断 12.34 已被精确存储。
