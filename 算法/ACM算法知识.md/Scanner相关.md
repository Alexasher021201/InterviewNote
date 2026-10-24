Java 写 ACM 模式算法题，除了算法本身，最需要掌握的是：**独立写程序、处理输入输出、使用常用数据结构。** `Scanner` 就属于输入工具。

可以按下面这个顺序学。

### 1. 程序的基本结构

ACM 模式通常要求自己读取输入、计算结果、输出答案，提交完整程序：

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        int a = sc.nextInt();
        int b = sc.nextInt();

        System.out.println(a + b);
    }
}
```

需要记住：

- 类名通常是 `Main`，以平台要求为准。
- 程序从 `main` 方法开始运行。
- 一般不要写 `package`。
- 输出只包含题目要求的答案，不要打印“请输入数字”等提示语。

### 2. 输入：先掌握 Scanner

常用方法：

| 方法 | 作用 | 输入示例 |
|---|---|---|
| `nextInt()` | 读取一个整数 | `123` |
| `nextLong()` | 读取一个长整数 | `10000000000` |
| `nextDouble()` | 读取一个小数 | `3.14` |
| `next()` | 读取一个字符串，遇到空白结束 | `hello` |
| `nextLine()` | 读取当前行剩余内容 | `hello world` |
| `hasNextInt()` | 判断后面是否还有整数 | 常用于读到输入结束 |

**`nextInt()`、`next()` 等默认按空白分隔，空格和换行都可以。**

例如输入：

```text
3
10 20 30
```

读取数组：

```java
int n = sc.nextInt();
int[] a = new int[n];

for (int i = 0; i < n; i++) {
    a[i] = sc.nextInt();
}
```

注意一个常见坑：**`nextInt()` 后直接调用 `nextLine()`，可能读到空字符串**，因为换行符还没有被消费。

```java
int n = sc.nextInt();
sc.nextLine();              // 消费这一行剩余内容
String s = sc.nextLine();    // 读取下一整行
```

### 3. 三种常见输入格式

**① 单组数据**

```java
int a = sc.nextInt();
int b = sc.nextInt();
System.out.println(a + b);
```

**② 第一行给出测试组数 T**

```java
int t = sc.nextInt();

while (t-- > 0) {
    int a = sc.nextInt();
    int b = sc.nextInt();
    System.out.println(a + b);
}
```

**③ 多组数据，读到输入结束 EOF**

```java
while (sc.hasNextInt()) {
    int a = sc.nextInt();
    int b = sc.nextInt();
    System.out.println(a + b);
}
```

具体用哪一种，要看题目的输入说明。

### 4. 输出：println 和 StringBuilder

基础输出：

```java
System.out.print(a);      // 不换行
System.out.println(a);    // 换行
System.out.printf("%.2f%n", x); // 保留两位小数
```

大量输出时，可以先拼接，再一次性打印：

```java
StringBuilder sb = new StringBuilder();

for (int i = 0; i < n; i++) {
    sb.append(a[i]).append('\n');
}

System.out.print(sb);
```

数据量大时，`Scanner` 可能较慢，之后再学习 **`BufferedReader + StringTokenizer`**，以及基于字节缓冲的快读模板。入门先把 `Scanner` 用熟即可。

### 5. Java 基础语法和常用 API

下面这些是算法题中最常用的：

| 知识 | 需要掌握的内容 |
|---|---|
| 基本类型 | `int`、`long`、`double`、`char`、`boolean` |
| 流程控制 | `if`、`for`、`while`、`break`、`continue` |
| 数组 | 一维数组、二维数组、遍历 |
| 方法 | 参数、返回值、递归 |
| 字符串 | `length()`、`charAt()`、`substring()`、`toCharArray()` |
| 数组工具 | `Arrays.sort()`、`Arrays.fill()` |
| 数学工具 | `Math.max()`、`Math.min()`、`Math.abs()` |
| 动态数组 | `ArrayList` |
| 映射与集合 | `HashMap`、`HashSet` |
| 栈与队列 | `ArrayDeque` |
| 优先队列 | `PriorityQueue` |

不需要一开始就全部背下来，可以做到哪类题再熟悉哪类工具。

### 6. 特别容易出错的几个地方

**整数溢出：**

```java
int a = 100000;
int b = 100000;

long ans = (long) a * b; // 先转 long，再乘
```

仅写 `long ans = a * b;`，乘法仍然按 `int` 计算，可能已经溢出。

**字符串比较：**

```java
s1.equals(s2) // 比较内容
```

不要用 `==` 判断字符串内容是否相同。

**整数除法：**

```java
5 / 2          // 2
(double) 5 / 2 // 2.5
```

**数组和字符串的长度写法不同：**

```java
a.length
s.length()
list.size()
```

**建议学习顺序：基本语法 → Scanner 输入输出 → 数组与字符串 → 排序 → 集合 → 快读。** 学到数组和字符串后，就可以开始做简单的 ACM 模式题了。