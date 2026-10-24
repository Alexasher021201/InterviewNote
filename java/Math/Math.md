# Java Math 复习笔记

> 重点：常用数学方法、除法结果保留一位小数、floor 与 ceil、常见面试陷阱。
> 示例默认使用普通范围内的数值；除法示例要求除数不为 0。

## 1. Math 是什么

`Math` 是 Java 自带的数学工具类，位于 `java.lang` 包，不需要手动导包，也不需要创建对象。

```java
int a = Math.abs(-5);       // 5
int b = Math.max(3, 8);     // 8
double c = Math.sqrt(9);    // 3.0
```

调用形式：`Math.方法名(参数)`。

## 2. 常用方法速查

| 方法 | 作用 | 示例 | 结果 |
| --- | --- | --- | --- |
| `Math.abs(x)` | 绝对值 | `Math.abs(-8)` | `8` |
| `Math.max(a, b)` | 较大值 | `Math.max(3, 7)` | `7` |
| `Math.min(a, b)` | 较小值 | `Math.min(3, 7)` | `3` |
| `Math.floor(x)` | 向下取整 | `Math.floor(3.8)` | `3.0` |
| `Math.ceil(x)` | 向上取整 | `Math.ceil(3.2)` | `4.0` |
| `Math.round(x)` | 取最近的整数，中点取更大的整数 | `Math.round(3.6)` | `4` |
| `Math.sqrt(x)` | 平方根 | `Math.sqrt(9)` | `3.0` |
| `Math.cbrt(x)` | 立方根 | `Math.cbrt(27)` | `3.0` |
| `Math.pow(a, b)` | a 的 b 次方 | `Math.pow(2, 3)` | `8.0` |
| `Math.random()` | `[0.0, 1.0)` 内的伪随机数 | `Math.random()` | 包含 0，不包含 1 |

常用常量：`Math.PI` 是圆周率，`Math.E` 是自然常数 e。

## 3. floor、ceil、round 和强制转换

### 3.1 floor：向下取整

取“不大于 x 的最大整数”，方向是往更小的数取。floor 可以理解为“地板”。

```java
Math.floor(3.2);  // 3.0
Math.floor(3.9);  // 3.0
Math.floor(-3.2); // -4.0
Math.floor(3.0);  // 3.0
```

### 3.2 ceil：向上取整

取“不小于 x 的最小整数”，方向是往更大的数取。ceil 可以理解为“天花板”。

```java
Math.ceil(3.2);  // 4.0
Math.ceil(3.9);  // 4.0
Math.ceil(-3.2); // -3.0
Math.ceil(3.0);  // 3.0
```

负数的位置：

```text
更小 ←──── -4 ───── -3.2 ───── -3 ────→ 更大
           floor               ceil
```

**注意：向下不是去掉小数，向上也不是无条件加一。原本就是整数时，两者都保持原值。**

`floor` 和 `ceil` 返回 `double`，即使结果的数值是整数。结果在 int 范围内时可以转换：

```java
int count = (int) Math.ceil(3.2); // 4
```

### 3.3 round：取最近的整数

两边一样近时，取更大的整数，也就是向正无穷方向取。

```java
Math.round(3.2);  // 3
Math.round(3.8);  // 4
Math.round(1.5);  // 2
Math.round(-1.5); // -1，注意不是 -2
```

返回类型：

```java
long a = Math.round(3.6); // double 参数，返回 long
int b = Math.round(3.6f); // float 参数，返回 int
```

### 3.4 `(int)`：去掉小数部分

对于处于 int 范围内的普通有限小数，转换成 int 是向零截断，不是四舍五入。

```java
int a = (int) 3.9;  // 3
int b = (int) -3.9; // -3
```

### 3.5 对照表

| x | floor(x) | ceil(x) | round(x) | (int) x |
| --- | --- | --- | --- | --- |
| `3.2` | `3.0` | `4.0` | `3` | `3` |
| `3.8` | `3.0` | `4.0` | `4` | `3` |
| `-3.2` | `-4.0` | `-3.0` | `-3` | `-3` |
| `-3.8` | `-4.0` | `-3.0` | `-4` | `-3` |
| `-1.5` | `-2.0` | `-1.0` | `-1` | `-1` |

## 4. 除法：先看操作数类型

### 4.1 两边都是整数：整数除法

```java
System.out.println(5 / 2); // 2
System.out.println(2 / 3); // 0
```

整数除法向零截断，不保留小数。

### 4.2 有一边是 double：浮点数除法

```java
System.out.println(5 / 2.0);        // 2.5
System.out.println((double) 2 / 3); // 0.6666666666666666
```

a、b 是整数时，要在除法发生之前转换：

```java
double result = (double) a / b;
```

常见错误：

```java
double x = 2 / 3;           // 0.0：先算出整数 0，再赋值
double y = (double) (2 / 3); // 0.0：除完再转已经来不及
double z = (double) 2 / 3;  // 正确：先转再除
```

### 4.3 10.0 和 10.00000 没有区别

```java
System.out.println(1 / 10);       // 0
System.out.println(1 / 10.0);     // 0.1
System.out.println(1 / 10.00000); // 0.1
```

`10.0` 和 `10.00000` 都是 double，数值相同。零的数量不会决定结果显示几位小数。

## 5. 面试题：除法结果四舍五入，保留一位小数

### 5.1 普通算法题，返回 double

```java
public static double divide(int a, int b) {
    double result = (double) a / b;
    return Math.round(result * 10) / 10.0;
}
```

示例：

```java
divide(2, 3);  // 0.7
divide(10, 3); // 3.3
divide(8, 3);  // 2.7
```

以 2 ÷ 3 为例：

```text
浮点数除法     0.6666...
乘以 10        6.6666...
round 取整     7
除以 10.0      0.7
```

**为什么最后必须用 10.0，而不是 10？**

因为 round(double) 返回 long，再除以整数 10 会发生整数除法：

```java
System.out.println(Math.round(6.666) / 10);   // 0
System.out.println(Math.round(6.666) / 10.0); // 0.7
```

**为什么结果到十分位？**

因为 round 已经得到整数，整数再除以十，得到十分之几。不是任何小数除以 10.0 都只剩一位小数。

```text
7 / 10.0     → 0.7
12 / 10.0    → 1.2
7.123 / 10.0 → 数学结果为 0.7123，仍有多位小数
```

推广到保留两位：

```java
double result = Math.round(x * 100) / 100.0;
```

这种写法受浮点精度和数值范围限制；负数中点遵循 Math.round 的规则。例如 x 为 -1.25 时，上述保留一位的写法得到 -1.2。如果要求严格十进制四舍五入，使用第 7 节。

## 6. 数值和显示格式是两件事

数学上，`0.7`、`0.70`、`0.700000` 是同一个数。

double 不记录“你想显示几位小数”。同一个变量可以按不同格式输出：

```java
double x = 0.7;

System.out.println(x); // 0.7
System.out.printf(java.util.Locale.ROOT, "%.1f%n", x); // 0.7
System.out.printf(java.util.Locale.ROOT, "%.5f%n", x); // 0.70000
```

格式含义：

- `%f`：按小数形式输出浮点数。
- `.1`：小数点后显示一位；`.5`：显示五位。
- `%n`：换行。
- `Locale.ROOT`：避免默认地区设置影响小数点符号。

如果只要求打印除法结果：

```java
System.out.printf(java.util.Locale.ROOT, "%.1f%n", (double) a / b);
```

如果要求返回固定一位小数的字符串：

```java
import java.util.Locale;

public static String divideToString(int a, int b) {
    return String.format(Locale.ROOT, "%.1f", (double) a / b);
}
```

格式化会舍入，并在必要时补零。严格来说，double 使用二进制存储，0.7 等许多十进制小数保存的是近似值。

## 7. 严格十进制四舍五入：BigDecimal

BigDecimal 不属于 Math，但适合需要明确控制十进制舍入规则的题目。

### 7.1 两个整数相除，直接保留一位

```java
import java.math.BigDecimal;
import java.math.RoundingMode;

public static BigDecimal divideDecimal(int a, int b) {
    return BigDecimal.valueOf(a)
            .divide(BigDecimal.valueOf(b), 1, RoundingMode.HALF_UP);
}
```

参数含义：`divide(除数, 保留的小数位数, 舍入规则)`。

```java
divideDecimal(2, 3);  // 0.7
divideDecimal(5, 4);  // 1.3
divideDecimal(-5, 4); // -1.3
divideDecimal(4, 2);  // 2.0
```

直接从整数构造并执行十进制除法，避免先计算 double 商再转换时带入的误差。

如果方法必须返回 double，可对结果调用 `.doubleValue()`；转换后仍是二进制浮点近似值，固定显示位数仍需格式化。

### 7.2 对已有十进制数调整小数位数

```java
BigDecimal x = new BigDecimal("3.15");
BigDecimal result = x.setScale(1, RoundingMode.HALF_UP); // 3.2
```

| 舍入规则 | 含义 | 3.15 保留一位 | -3.15 保留一位 |
| --- | --- | --- | --- |
| `HALF_UP` | 取最近值，中点向远离零的方向取 | `3.2` | `-3.2` |
| `DOWN` | 向零截断 | `3.1` | `-3.1` |
| `FLOOR` | 向负无穷方向取 | `3.1` | `-3.2` |
| `CEILING` | 向正无穷方向取 | `3.2` | `-3.1` |

构造时：

- 精确十进制常量优先用 `new BigDecimal("3.15")`。
- 已有 double x 时，常用 `BigDecimal.valueOf(x)`，但无法消除之前计算的误差。
- 不要用 `new BigDecimal(3.15)` 表达精确十进制 3.15，它会保留该 double 的二进制近似值。
- BigDecimal 不可变，setScale 返回新对象，需要接收返回值。

## 8. 常见做题场景

### 8.1 需要几辆车、几页：ceil

23 人，每辆车坐 5 人，最后一辆即使没坐满也需要一辆。

```java
int people = 23;
int capacity = 5;
int cars = (int) Math.ceil((double) people / capacity); // 5
```

错误写法：`Math.ceil(people / capacity)`，整数除法先得到 4，ceil 的结果只能是 4.0。

### 8.2 绝对差值：abs

```java
int a = 3;
int b = 8;
int distance = Math.abs(a - b); // 5
```

如果 a、b 可能取任意 int 值，在减法前提升为 long：

```java
long distance = Math.abs((long) a - b);
```

边界：`Math.abs(Integer.MIN_VALUE)` 仍是负数，因为对应正数超出 int 范围。

### 8.3 更新最大值、最小值

```java
int[] nums = {3, 8, 2, 5};
int largest = nums[0];
int smallest = nums[0];

for (int num : nums) {
    largest = Math.max(largest, num);
    smallest = Math.min(smallest, num);
}
// largest = 8，smallest = 2；这里要求数组非空
```

### 8.4 乘方和开方

```java
int x = 5;
int square = x * x;            // 25，整数乘法要留意溢出
double power = Math.pow(2, 3); // 8.0
double root = Math.sqrt(9);    // 3.0
```

Java 的 `^` 是按位异或，不是乘方：`2 ^ 3` 是 1，`Math.pow(2, 3)` 才是 8.0。

Math.pow 返回 double；整数平方通常直接写 x * x 更清楚。

### 8.5 随机整数

```java
int x = (int) (Math.random() * 10);     // 0～9
int y = (int) (Math.random() * 10) + 1; // 1～10
```

理解：`[0, 1)` → 乘 10 得到 `[0, 10)` → 转整数得到 0～9 → 加 1 得到 1～10。

注意括号：`(int) Math.random() * 10` 总是 0，因为先将小于 1 的非负数转成了整数 0。

## 9. 低频方法：先认识用途

| 方法 | 用途 |
| --- | --- |
| `Math.log(x)` | 自然对数，以 e 为底 |
| `Math.log10(x)` | 以 10 为底的对数 |
| `Math.exp(x)` | e 的 x 次方 |
| `Math.sin(x)`、`cos(x)`、`tan(x)` | 三角函数，参数单位是弧度 |
| `Math.toRadians(degrees)` | 角度转弧度 |
| `Math.toDegrees(radians)` | 弧度转角度 |

```java
double value = Math.sin(Math.toRadians(30)); // 约 0.5
```

优先掌握前面的高频方法，低频方法按题目需要查阅。

## 10. 自测题

先判断结果，再查看答案。前九题判断表达式或变量的值，第十题判断输出内容。

```java
// 1
Math.floor(-2.3)
// 2
Math.ceil(-2.3)
// 3
Math.round(-2.5)
// 4
(int) -2.9
// 5
double x = 5 / 2;
// 6
double y = (double) 5 / 2;
// 7
Math.round(((double) 2 / 3) * 10) / 10.0
// 8
Math.round(((double) 2 / 3) * 10) / 10
// 9
Math.ceil(23 / 5)
// 10
System.out.printf(java.util.Locale.ROOT, "%.3f%n", 1 / 10.00000);
```

<details>
<summary>展开答案</summary>

| 题号 | 答案 | 原因 |
| --- | --- | --- |
| 1 | `-3.0` | floor 向更小的方向取整 |
| 2 | `-2.0` | ceil 向更大的方向取整 |
| 3 | `-2`，long 类型 | round 中点取更大的整数 |
| 4 | `-2` | 强制转换向零截断 |
| 5 | x 为 `2.0` | 先做整数除法，再赋值 |
| 6 | y 为 `2.5` | 除法前已转换为 double |
| 7 | `0.7` | 放大、取整，再用浮点数除法缩小 |
| 8 | `0`，long 类型 | 最后发生整数除法 |
| 9 | `4.0` | 23 / 5 先得到整数 4 |
| 10 | `0.100`，然后换行 | 显示三位由 %.3f 控制 |

</details>

## 11. 面试前快速回顾

- floor 往更小取，ceil 往更大取；原本就是整数则保持原值。
- round 取最近的整数；中点取更大的整数，尤其留意负数。
- 转 int 对普通范围内的有限小数是向零截断。
- 要小数，除法前转换类型；除完再转可能已经丢失小数。
- 普通算法题保留一位：`Math.round(x * 10) / 10.0`。
- 严格十进制四舍五入：BigDecimal 配合 HALF_UP。
- 固定显示一位：`%.1f`；显示位数不由 10.00000 的零决定。
- Math.pow 是乘方，^ 是按位异或。

## 参考资料

- [Java Math 官方文档](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Math.html)
- [Java Formatter 官方文档](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Formatter.html)
- [Java BigDecimal 官方文档](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/math/BigDecimal.html)
