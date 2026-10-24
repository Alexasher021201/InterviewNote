# Java String 常用方法复习笔记

> 范围：从你提供的菜鸟教程 String 页面中，挑选日常编程与刷题常用的方法，按使用场景整理；额外补充少量常用 API，并标注版本。
>
> 每节包含用途、示例结果和易错点。重点复习 `charAt`、`substring`、`equals`、`indexOf`、`split`、`replace`、`toCharArray`。
>
> 示例主要适用于 Java 8+；`strip`、`isBlank`、`repeat` 为 Java 11+ 补充。代码中的 `//` 是预期结果或解释，示例片段按节独立阅读。

## 1. 高频方法速查

除标注“静态”的方法外，通常用 `s.方法(...)` 调用。

| 方法 | 返回类型 | 用途 | 首要易错点 |
| --- | --- | --- | --- |
| `length()` | `int` | 获取长度 | 字符串是方法，数组是 `length` 属性 |
| `isEmpty()` | `boolean` | 长度是否为 0 | `" "` 不为空；null 不能调用 |
| `charAt(i)` | `char` | 读取下标 i 的字符 | 最大下标是 `length() - 1` |
| `substring(begin)` | `String` | 从 begin 截取到末尾 | 不会自动修改原变量 |
| `substring(begin, end)` | `String` | 截取 `[begin, end)` | 不包含 end |
| `equals(obj)` | `boolean` | 比较内容 | 字符串内容不要用 `==` 比 |
| `equalsIgnoreCase(s)` | `boolean` | 忽略大小写比较内容 | 空格仍参与比较 |
| `compareTo(s)` | `int` | 按字典顺序比较 | 只看正负与 0，不保证只返回 ±1 |
| `compareToIgnoreCase(s)` | `int` | 忽略大小写比较顺序 | 不是数值比较 |
| `contains(text)` | `boolean` | 是否包含文字片段 | 参数不是正则；字符要写成字符串 |
| `indexOf(ch/text[, from])` | `int` | 从前查找 | 找不到是 `-1`，找到下标 0 也算成功 |
| `lastIndexOf(ch/text[, from])` | `int` | 从后查找 | from 指定反向查找的起点 |
| `startsWith(prefix[, offset])` | `boolean` | 检查前缀 | 区分大小写 |
| `endsWith(suffix)` | `boolean` | 检查后缀 | 区分大小写 |
| `toLowerCase([locale])` | `String` | 整段转小写 | 接住返回值，注意语言环境 |
| `toUpperCase([locale])` | `String` | 整段转大写 | 长度可能变化 |
| `trim()` | `String` | 去掉两端特定字符 | 不删除中间空格，也不覆盖所有 Unicode 空白 |
| `replace(old, replacement)` | `String` | 按字面量替换全部匹配 | 可以替换全部，不只替换第一个 |
| `replaceAll(regex, replacement)` | `String` | 按正则替换全部匹配 | 正则转义、替换串中的 `$` 与 `\` |
| `replaceFirst(regex, replacement)` | `String` | 按正则替换首次匹配 | 参数也会按正则解释 |
| `split(regex[, limit])` | `String[]` | 按正则拆分 | 默认丢掉末尾空串 |
| `matches(regex)` | `boolean` | 整个字符串是否匹配 | 不是“某一部分是否匹配” |
| `toCharArray()` | `char[]` | 转为字符数组 | 修改数组不会修改原字符串 |
| `String.valueOf(value)`，静态 | `String` | 转成字符串 | 不是字符串转整数 |
| `String.copyValueOf(chars[, offset, count])`，静态 | `String` | 从字符数组构造字符串 | count 是数量，区别于 substring 的 end |
| `concat(s)` | `String` | 拼接两个字符串 | 接住返回值 |
| `String.format(format, args...)`，静态 | `String` | 格式化字符串 | 占位符要与参数类型匹配 |
| `getBytes(charset)` | `byte[]` | 按字符集编码 | 明确指定编码 |

补充：`String.join(...)` 从 Java 8 起提供；`strip()`、`isBlank()`、`repeat()` 从 Java 11 起提供。

## 2. 必须先记住：String 不可变

```java
String s = "Abc";
s.toLowerCase();
System.out.println(s); // Abc

s = s.toLowerCase();
System.out.println(s); // abc
```

转换、截取、替换、拼接等操作不会修改原 String 对象。想让变量指向处理后的结果，需要赋值。

不可变不等于“变量不能重新赋值”：

```java
String a = "abc";
String b = a;
a = a.substring(1);

System.out.println(a); // bc
System.out.println(b); // abc
```

具体方法在某些情况下可能返回原对象；不要把“内容处理”理解成“保证创建新对象”。复习时关注返回的内容，并使用 `equals` 比较。

## 3. length、isEmpty：长度与空字符串

```java
"abc".length();  // 3
"".length();     // 0
"".isEmpty();    // true
" ".isEmpty();   // false：有一个空格
"\t".isEmpty();  // false：有一个制表符
```

不要混用这三种写法：

```java
int n1 = s.length();       // String
int n2 = new int[3].length; // 数组
int n3 = new java.util.ArrayList<Integer>().size(); // 集合
```

### null 和空字符串不同

```java
String a = null; // 没有指向字符串对象
String b = "";   // 指向长度为 0 的字符串

boolean empty = a == null || a.isEmpty(); // 安全：短路求值
// a.isEmpty(); // NullPointerException
```

### Java 11+：isBlank

```java
"".isBlank();      // true
" \t\n".isBlank(); // true
" a ".isBlank();   // false
```

`isBlank()` 检查空串或全部由 Java 空白字符组成的字符串；`isEmpty()` 只检查长度为 0。Java 的空白定义并不包含所有 Unicode 空格，例如 NBSP。

## 4. charAt：按下标取字符

```java
String s = "java";
char first = s.charAt(0);              // 'j'
char last = s.charAt(s.length() - 1);   // 'a'
```

合法范围：`0 <= index < s.length()`。

```java
// s.charAt(s.length()); // 越界
// "".charAt(0);        // 越界
```

在循环里，应先判断边界，再取字符：

```java
int i = 0;
while (i < s.length() && s.charAt(i) == ' ') {
    i++;
}
```

Java 的 `&&` 从左向右短路求值，左边为假时不会执行右边。

## 5. substring：截取字符串

```java
String s = "abcdef";
s.substring(2);    // "cdef"
s.substring(1, 4); // "bcd"
s.substring(3, 3); // ""
s.substring(6);    // ""：起点允许等于 length()
```

双参数规则：**左闭右开 `[begin, end)`**。

```text
字符：  a b c d e f
下标：  0 1 2 3 4 5

substring(1, 4) 保留下标 1、2、3，不包含 4。
结果长度为 4 - 1 = 3。
```

合法条件：`0 <= begin <= end <= s.length()`。

### 和数组构造函数的参数区别

```java
char[] chars = {'a', 'b', 'c', 'd', 'e'};
String a = new String(chars, 1, 3);       // "bcd"：从 1 开始，取 3 个
String b = String.copyValueOf(chars, 1, 3); // "bcd"：从 1 开始，取 3 个
String c = "abcde".substring(1, 3);       // "bc"：结束下标是 3
```

`offset, count` 是“起点 + 数量”；`begin, end` 是“起点 + 结束下标”。

### 你在第 8 题中犯过的错误：遍历时反复截取

```java
String str = "42";
for (int j = 0; j < str.length(); j++) {
    if (str.charAt(j) != '0') {
        str = str.substring(j);
    }
}
// 最终 str 是 "2"
```

`j` 在增加，str 的内容和长度也在变化，下标便不再对应原来的字符串。

更清楚的做法是：先在原串上移动下标，确定位置后再截取一次：

```java
int i = 0;
while (i < s.length() && s.charAt(i) == ' ') {
    i++;
}
String rest = s.substring(i); // 全是空格时也合法，结果为 ""
```

如果只是把数字字符逐位累积成整数，前导零可以直接参与计算，通常不需要先删除。

## 6. equals、equalsIgnoreCase：比较内容

```java
String a = new String("java");
String b = new String("java");

System.out.println(a == b);      // false：比较是不是同一个对象
System.out.println(a.equals(b)); // true：比较内容

"Java".equals("java");           // false
"Java".equalsIgnoreCase("java"); // true
"java ".equalsIgnoreCase("java"); // false：空格仍然参与比较
```

**易错：有些字面量使用 `==` 会恰好得到 true，不代表这种内容比较方式可靠。**

避免空指针的常用写法：

```java
String s = null;
boolean a = "ok".equals(s);               // false
boolean b = java.util.Objects.equals(s, null); // true
```

`Objects.equals(a, b)` 适合两个变量都可能为 null 的场景。`equalsIgnoreCase` 适合常见大小写忽略比较，不是语言相关排序规则。

## 7. compareTo、compareToIgnoreCase：比较字典顺序

```java
"abc".compareTo("abd"); // -1
"abc".compareTo("abc"); // 0
"z".compareTo("a");     // 25，不是 1
"ab".compareTo("abc");  // -1：公共前缀相同，短的在前
"Java".compareToIgnoreCase("java"); // 0
```

记住：小于 0 表示当前字符串在前；等于 0 表示比较结果相同；大于 0 表示在后。

**易错：按字符顺序比较，不是按数字大小比较。**

```java
"10".compareTo("2"); // 小于 0，因为第一位 '1' 小于 '2'
```

自然语言排序（例如中文拼音排序）需要专门的规则，不能直接依赖 `compareTo`。

## 8. contains、indexOf、lastIndexOf：包含与查找

```java
String s = "banana";

s.contains("ana");       // true
s.indexOf('a');          // 1
s.indexOf('a', 2);       // 3：从下标 2 开始向右找，包含 2
s.indexOf("ana");       // 1
s.indexOf("ana", 2);    // 3
s.lastIndexOf('a');      // 5
s.lastIndexOf('a', 4);   // 3：从下标 4 开始向左找，包含 4
s.lastIndexOf("ana");   // 3
s.indexOf("xyz");       // -1
```

正确检查是否找到：

```java
if (s.indexOf("ban") >= 0) {
    // 找到了，下标可能是 0
}
```

不要写成 `> 0`，否则会漏掉出现在开头的匹配。

其他易错点：

- `contains`、`indexOf` 都区分大小写。
- `contains(".")` 查找的是字面量点号，不是正则中的“任意字符”。
- `contains` 接受字符序列：写 `s.contains("a")`，不能写 `s.contains('a')`；查单字符也可以用 `indexOf('a') >= 0`。
- `s.indexOf("")` 为 `0`；`s.lastIndexOf("")` 为 `s.length()`。
- `indexOf` 返回 `-1` 时，不要直接拿它作为 `charAt` 或 `substring` 的下标。

## 9. startsWith、endsWith：前缀与后缀

```java
"hello.java".startsWith("hello"); // true
"hello.java".startsWith("java", 6); // true：从下标 6 检查
"hello.java".endsWith(".java"); // true
"hello.java".endsWith(".JAVA"); // false
"abc".startsWith(""); // true
"abc".endsWith("");   // true
```

它们不会自动忽略大小写，也不会把参数解释成正则表达式。

## 10. toLowerCase、toUpperCase：大小写转换

```java
import java.util.Locale;

String lower = "Hello123".toLowerCase(Locale.ROOT); // "hello123"
String upper = "Hello123".toUpperCase(Locale.ROOT); // "HELLO123"
```

不传 Locale 时使用默认语言环境。处理程序标识、协议文字等希望结果不受系统语言影响的内容时，可显式使用 `Locale.ROOT`。

**易错：转换后长度可能变化。**

```java
"ß".toUpperCase(Locale.ROOT); // "SS"
```

只比较两个普通字符时，参看 Character 笔记中的 `Character.toLowerCase(c)`。

## 11. trim 与 strip：去掉两端空白

```java
"  a b  ".trim();  // "a b"：中间空格保留
"\t42\n".trim();  // "42"
"   ".trim();      // ""
```

`trim()` 去掉两端字符值不大于 `U+0020` 的字符。它不会删除所有 Unicode 空白。

Java 11+ 的 `strip()` 使用 Java 的空白字符定义，还提供两种单侧处理方法：

```java
"\u2003abc\u2003".strip(); // "abc"：EM SPACE
"  abc  ".stripLeading();  // "abc  "
"  abc  ".stripTrailing(); // "  abc"
```

| 需求 | 方法 |
| --- | --- |
| 删除两端普通空格、制表符等，兼容 Java 8 | `trim()` |
| 按 Java 空白定义删除两端空白，Java 11+ | `strip()` |
| 只删除开头空白，Java 11+ | `stripLeading()` |
| 删除字符串中所有普通空格 | `replace(" ", "")` |

注意 `strip()` 也不移除 NBSP：`"\u00A0".strip()` 仍有内容。不要笼统理解为“清理一切空白”。

## 12. replace、replaceAll、replaceFirst：替换

### 12.1 replace：按字面量替换，全部生效

```java
"banana".replace('a', 'o');     // "bonono"
"a.b.c".replace(".", "-");  // "a-b-c"
"abcabc".replace("ab", "X"); // "XcXc"
"a b c".replace(" ", "");    // "abc"
```

`replace` 有字符重载和字符序列重载；字符串重载也会替换所有匹配，不只替换第一个。

### 12.2 replaceAll：按正则替换全部

```java
"a1b22".replaceAll("[0-9]+", "#");  // "a#b#"
"A, b! 2".replaceAll("[^a-zA-Z0-9]", ""); // "Ab2"
"a.b".replaceAll(".", "X");        // "XXX"：. 在正则中匹配任意非行终止字符
"a.b".replaceAll("\\.", "-");     // "a-b"：转义后才是字面量点号
```

### 12.3 replaceFirst：按正则只替换第一次

```java
"a1b22".replaceFirst("[0-9]+", "#"); // "a#b22"
```

### 替换时最容易混淆的两层语法

1. 匹配模式里的正则特殊字符，需要按正则规则转义。
2. Java 字符串中的反斜杠，还需要按 Java 源码规则转义。

因此，正则要表达 `\.`，Java 源码应写 `"\\."`。

动态文本可以借助工具安全处理：

```java
String text = "a.b";
String search = ".";
String replacement = "$";

String answer = text.replaceAll(
    java.util.regex.Pattern.quote(search),
    java.util.regex.Matcher.quoteReplacement(replacement)
); // "a$b"
```

**易错：正则替换的 replacement 中，`$` 可表示捕获组引用，`\` 可用于转义。** 动态替换文本用 `Matcher.quoteReplacement`；纯文字替换优先考虑 `replace(search, replacement)`。

## 13. split：拆分字符串

```java
String[] a = "red,green,blue".split(",");
// ["red", "green", "blue"]

String[] b = "a.b.c".split("\\.");
// ["a", "b", "c"]

String[] c = "a|b|c".split("\\|");
// ["a", "b", "c"]
```

**参数是正则。** 点号和竖线等不能随意当作普通分隔符；动态分隔符可用 `Pattern.quote(delimiter)`。

### limit 决定拆分次数与末尾空串

```java
"a,b,,".split(",");     // ["a", "b"]：默认 limit = 0
"a,b,,".split(",", -1); // ["a", "b", "", ""]
"a,b,c".split(",", 2);  // ["a", "b,c"]：至多拆一次
"a,,b".split(",");     // ["a", "", "b"]：中间空串会保留
```

| limit | 行为 |
| --- | --- |
| `0`，默认 | 尽量拆分，删除末尾空字符串 |
| 负数，例如 `-1` | 尽量拆分，保留末尾空字符串 |
| 正数 `k` | 最多拆分 `k - 1` 次，剩余内容放在最后一项 |

### 连续空格、空字符串

```java
"a  b".split(" ");      // ["a", "", "b"]
"a  b\tc".split("\\s+"); // ["a", "b", "c"]
"".split(",");         // [""]：数组长度是 1
```

注意 `Arrays.toString(new String[]{""})` 看起来是 `[]`，容易误以为数组为空；检查 `length` 更可靠。

“按空白拆成单词”可以先处理空输入：

```java
String text = s.trim();
String[] words = text.isEmpty() ? new String[0] : text.split("\\s+");
```

这里默认正则 `\s` 的范围主要是 ASCII 空白；需要匹配 Unicode 空白时，应明确设置正则规则，例如 `(?U)\s+`。不要认为它与 `Character.isWhitespace` 的集合完全相同。

## 14. matches：整个字符串是否匹配

```java
"123".matches("[0-9]+");    // true
"abc123".matches("[0-9]+"); // false：不是整串都由数字组成
"".matches("[0-9]+");       // false：+ 表示至少一个
"".matches("[0-9]*");       // true：* 允许零个
```

只要判断是否包含固定文字，用 `contains`；要在任意位置查找正则匹配，用 `Matcher.find()`：

```java
boolean found = java.util.regex.Pattern.compile("[0-9]+")
    .matcher("abc123")
    .find(); // true
```

## 15. toCharArray、valueOf、copyValueOf：字符串与数组转换

```java
String s = "abc";
char[] chars = s.toCharArray();
chars[0] = 'X';

System.out.println(s); // abc：原字符串不变

String a = new String(chars);             // "Xbc"
String b = String.valueOf(chars);         // "Xbc"
String c = String.copyValueOf(chars);     // "Xbc"
String d = String.copyValueOf(chars, 1, 2); // "bc"
```

反向转换得到的字符串内容，也不会随着原数组后续修改而变化。

### 字符串与整数不要混淆方向

```java
String a = String.valueOf(123);   // "123"
String b = String.valueOf('7');   // "7"
String c = String.valueOf(true);  // "true"

int n = Integer.parseInt("123"); // 123：这是 Integer 的方法
```

`Integer.parseInt` 要求输入符合整数格式且不越界，失败会抛出 `NumberFormatException`。它不能直接实现第 8 题那种“读到非数字就停止，越界截断”的全部规则。

### null 与重载陷阱

```java
String a = String.valueOf((Object) null); // "null"
// String b = String.valueOf((char[]) null); // NullPointerException
// String c = String.valueOf(null); // 选择 char[] 重载，也会抛异常
```

不要笼统记成“valueOf 遇到 null 一定返回字符串 null”，还要看选中了哪个重载。

## 16. concat、加号、join：拼接

```java
"ab".concat("cd"); // "abcd"
String joined = "ab" + "cd"; // "abcd"
```

`+` 从左向右计算，字符串出现的位置会影响结果：

```java
System.out.println(1 + 2 + "3"); // 33：先算 1 + 2
System.out.println("1" + 2 + 3); // 123：一路拼接
```

### Java 8+ 补充：join

```java
String s = String.join("-", "a", "b", "c"); // "a-b-c"
String t = String.join(",", java.util.Arrays.asList("x", "y")); // "x,y"
```

### 循环里大量拼接，优先 StringBuilder

```java
StringBuilder builder = new StringBuilder();
for (int i = 0; i < 3; i++) {
    builder.append(i);
}
String answer = builder.toString(); // "012"
```

反复执行 `s = s + piece`，会不断构造更长的结果，常见情况下会产生大量重复复制。StringBuilder 适合循环累积。

String 没有 `reverse()`，常用写法是：

```java
String reversed = new StringBuilder("abc").reverse().toString(); // "cba"
```

## 17. format：按模板生成字符串

```java
String a = String.format(java.util.Locale.ROOT,
    "name=%s, count=%d", "Tom", 3); // "name=Tom, count=3"

String b = String.format(java.util.Locale.ROOT,
    "%.2f", 3.14159); // "3.14"

String c = String.format(java.util.Locale.ROOT,
    "%04d", 7); // "0007"

String d = String.format(java.util.Locale.ROOT,
    "进度=%d%%", 80); // "进度=80%"
```

| 占位符 | 用途 |
| --- | --- |
| `%s` | 字符串表示 |
| `%d` | 十进制整数 |
| `%f` | 浮点数 |
| `%.2f` | 保留两位小数 |
| `%c` | 字符 |
| `%%` | 一个百分号 |
| `%n` | 当前平台的换行符 |

格式与参数类型不匹配，例如 `%d` 配 `"7"`，会抛出格式化异常。

## 18. getBytes：字符串编码成字节

```java
import java.nio.charset.StandardCharsets;

String text = "中文";
byte[] bytes = text.getBytes(StandardCharsets.UTF_8);
String restored = new String(bytes, StandardCharsets.UTF_8);

System.out.println(text.length());  // 2
System.out.println(bytes.length);   // 6
System.out.println(restored);       // 中文
```

**易错：字符串长度不等于字节数。** 编码与解码应使用约定的字符集，不要依赖运行环境的默认值。

## 19. Java 11+ 补充：repeat

```java
"ab".repeat(3); // "ababab"
"ab".repeat(0); // ""
// "ab".repeat(-1); // IllegalArgumentException
```

## 20. Unicode：length 不一定是可见字符数量

```java
String emoji = "\uD83D\uDE00"; // 😀
emoji.length(); // 2
emoji.codePointCount(0, emoji.length()); // 1
```

String 的下标基于 UTF-16 代码单元。`charAt(0)` 在这个例子中只会返回代理对的一半。处理完整码点时，可以使用 `codePointAt` 或 `codePoints()`。

这与底层具体采用哪种存储优化无关。不要把 String 的实现固定记成“内部一定是 char 数组”；复习时关注 API 行为。

## 21. 按题目选择方法

| 场景 | 优先考虑 |
| --- | --- |
| 两边向中间比较回文 | `charAt` + Character 判断与大小写转换 |
| 从字符串中取一段 | `substring` |
| 判断字符串内容相同 | `equals` / `equalsIgnoreCase` |
| 查找某个字符的位置 | `indexOf` / `lastIndexOf` |
| 判断文件名后缀 | `endsWith` |
| 去掉两端普通空白 | `trim` |
| 删除或替换固定文字 | `replace` |
| 按模式过滤字符 | `replaceAll` |
| 按分隔符分组 | `split`，先确认分隔符是否有正则含义 |
| 对字符排序 | `toCharArray` → `Arrays.sort` → `new String(chars)` |
| 大量追加字符 | `StringBuilder.append` |
| 手写 atoi | `charAt` + 下标移动 + 数字判断 + 整数边界检查 |

## 22. 复习自测与答案

1. `"abcd".substring(1, 3)`？—— `"bc"`。
2. `"".charAt(0)`？—— 越界异常。
3. `" ".isEmpty()`？—— `false`。
4. `s.toLowerCase()` 后，s 一定变小写吗？—— 不会自动修改变量，需要接住返回值。
5. `indexOf` 返回 0 是找到了吗？—— 是，下标从 0 开始。
6. `replace(".", "-")` 和 `replaceAll(".", "-")` 一样吗？—— 不一样，后者把点号当正则。
7. 按字面量点号拆分？—— `s.split("\\.")`。
8. 拆分时保留末尾空项？—— `s.split(",", -1)`。
9. `"10".compareTo("2")` 的结果是正还是负？—— 负，按字符顺序比较。
10. `"abc123".matches("[0-9]+")`？—— `false`，要求整串匹配。
11. 修改 `toCharArray()` 返回的数组，会改变 s 吗？—— 不会。
12. `String.valueOf(123)` 和 `Integer.parseInt("123")` 的方向？—— 前者转字符串，后者转整数。

## 参考来源与范围说明

- 选题依据：[菜鸟教程：Java String 类](https://www.runoob.com/java/java-string.html)。本笔记使用自行编写的例子与分类，精选常用 API。
- 方法签名、边界与版本：[Java SE 17 String 官方 API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html)。
- 字符判断配套资料：[Java Character 复习笔记](Java_Character_复习笔记.md)。

网页列表中未重点展开的低频方法，例如 `intern`、`getChars`、`regionMatches`、`hashCode` 等，可在需要时查官方文档。对 `compareTo`，常规 String 代码使用 `compareTo(String)`，不需要把网页中的 Object 条目另当成一种常用写法。
