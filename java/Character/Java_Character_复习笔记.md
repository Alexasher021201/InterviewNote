# Java Character 复习笔记

> 阅读方式：先看速查表，做题遇到问题时再查对应示例与易错点。
>
> 范围：完整覆盖你提供的菜鸟教程页面「Character 方法」表中的 **8 个方法**，另外补充常用方法。这里的“全部”指该页面列出的方法，不是整个 JDK 的 Character API。
>
> 示例以 Java 8+ 常见用法为主；版本有要求的补充内容会单独标注。

## 1. 先分清 char、Character、String

| 类型 | 用途 | 示例 |
| --- | --- | --- |
| `char` | 基本类型，保存一个 UTF-16 代码单元 | `char c = 'a';` |
| `Character` | `char` 的包装类，并提供字符工具方法 | `Character c = 'a';` |
| `String` | 字符串对象，可以为空，也可以包含多个字符 | `String s = "abc";` |

单个字符使用单引号，字符串使用双引号：

```java
char c = '7';
String s = "7";
int n = 7;
```

三者分别是字符、字符串、整数，不是同一种类型。`char` 和 `Character` 属于 `java.lang`，通常不需要额外导包。

下面介绍的字符判断与转换方法通常直接通过类名调用：

```java
boolean ok = Character.isDigit(c);
char lower = Character.toLowerCase(c);
```

## 2. 原网页全部 8 个方法速查

| 方法：常用签名 | 返回类型 | 用途 | 示例及结果 |
| --- | --- | --- | --- |
| `Character.isLetter(char c)` | `boolean` | 判断是否为字母 | `isLetter('A')` → `true` |
| `Character.isDigit(char c)` | `boolean` | 判断是否为十进制数字字符 | `isDigit('7')` → `true` |
| `Character.isWhitespace(char c)` | `boolean` | 判断是否为 Java 定义的空白字符 | `isWhitespace('\n')` → `true` |
| `Character.isUpperCase(char c)` | `boolean` | 判断是否为大写字符 | `isUpperCase('A')` → `true` |
| `Character.isLowerCase(char c)` | `boolean` | 判断是否为小写字符 | `isLowerCase('a')` → `true` |
| `Character.toUpperCase(char c)` | `char` | 获取大写形式 | `toUpperCase('a')` → `'A'` |
| `Character.toLowerCase(char c)` | `char` | 获取小写形式 | `toLowerCase('A')` → `'a'` |
| `Character.toString(char c)` | `String` | 将一个 `char` 转成字符串 | `toString('a')` → `"a"` |
| `Character.isLetterOrDigit(char c)` | `boolean` | 判断一个字符是否是字母或数字 | `Character.isLetterOrDigit(`+`)` → `false` |

表内示例省略了前缀 `Character.`。这 8 个方法的具体说明如下。

## 3. 判断类方法：返回 boolean

### 3.1 isLetter：是不是字母

```java
Character.isLetter('A'); // true
Character.isLetter('z'); // true
Character.isLetter('中'); // true
Character.isLetter('7'); // false
Character.isLetter('_'); // false
```

**易错：它依据 Unicode 判断，不只识别英文字母。** 如果题目要求只接受英文，可以明确写范围：

```java
boolean english = (c >= 'a' && c <= 'z')
               || (c >= 'A' && c <= 'Z');
```

### 3.2 isDigit：是不是数字字符

```java
Character.isDigit('0'); // true
Character.isDigit('9'); // true
Character.isDigit('７'); // true：全角数字
Character.isDigit('a'); // false
Character.isDigit('-'); // false
Character.isDigit('.'); // false
```

它只判断一个字符，不能直接判断字符串是不是整数：

```java
// Character.isDigit("123"); // 编译错误：参数不是 String
```

**易错：`'-'`、`'+'`、`'.'` 都不是数字字符。** 因此解析负数、小数时，符号和小数点需要单独处理。

做只接受 `'0'`～`'9'` 的题目时，可以使用：

```java
boolean digit = c >= '0' && c <= '9';
```

### 3.3 isWhitespace：是不是空白字符

```java
Character.isWhitespace(' ');  // true：普通空格
Character.isWhitespace('\t'); // true：制表符
Character.isWhitespace('\n'); // true：换行符
Character.isWhitespace('\r'); // true：回车符
Character.isWhitespace('a');  // false
```

**易错：判断普通空格和判断空白字符，范围不同。**

```java
c == ' ';                  // 只检查普通空格
Character.isWhitespace(c); // 还包括制表、换行等 Java 空白字符
```

按题目要求选择。例如只允许跳过普通空格时，不要擅自扩展范围。

它也不是“所有看上去像空格的 Unicode 字符”：

```java
Character.isWhitespace('\u00A0'); // false：不换行空格 NBSP
Character.isSpaceChar('\u00A0');  // true：补充方法，判断 Unicode 空格类别
```

### 3.4 isUpperCase：是不是大写字符

```java
Character.isUpperCase('A'); // true
Character.isUpperCase('a'); // false
Character.isUpperCase('7'); // false
```

### 3.5 isLowerCase：是不是小写字符

```java
Character.isLowerCase('a'); // true
Character.isLowerCase('A'); // false
Character.isLowerCase('7'); // false
```

**易错：不是大写，不代表就是小写。** 数字、标点等，两个判断都可能返回 `false`。

## 4. 转换类方法：接住返回值

### 4.1 toUpperCase：转为大写

```java
char c = 'a';
Character.toUpperCase(c);
System.out.println(c); // a：上面没有接住返回值

c = Character.toUpperCase(c);
System.out.println(c); // A

Character.toUpperCase('A'); // 'A'
Character.toUpperCase('7'); // '7'
```

### 4.2 toLowerCase：转为小写

```java
char c = Character.toLowerCase('A'); // 'a'

boolean same = Character.toLowerCase('A')
            == Character.toLowerCase('a'); // true
```

适合忽略大小写比较单个字符。没有相应大小写转换的字符通常保持原值。

**易错：Character 的单字符转换，不能完整替代 String 的整段大小写转换。** 某些语言中，一个字符转大写后会成为多个字符：

```java
String upper = "ß".toUpperCase(java.util.Locale.ROOT); // "SS"
```

`Character.toUpperCase(char)` 返回一个 `char`，无法返回两个字符的 `"SS"`。

### 4.3 toString：字符转字符串

```java
String a = Character.toString('7'); // "7"
String b = String.valueOf('7');     // "7"，也很常用

Character boxed = 'a';
String c = boxed.toString();       // "a"：实例方法，无参数
```

记住：

```java
Character.toString('7'); // String "7"
Character.valueOf('7');  // Character 对象，里面装着字符 '7'
```

两者返回类型不同。

## 5. 做题常用补充：原网页 8 项之外

### 5.1 isLetterOrDigit：字母或数字

```java
Character.isLetterOrDigit('A'); // true
Character.isLetterOrDigit('7'); // true
Character.isLetterOrDigit(','); // false
Character.isLetterOrDigit(' '); // false
```

相当于判断“是字母，或者是数字”。适合过滤标点的题目。注意它同样按 Unicode 定义识别。

### 5.2 digit：把指定进制中的数字字符转为整数

```java
Character.digit('7', 10); // 7
Character.digit('７', 10); // 7
Character.digit('F', 16); // 15
Character.digit('x', 10); // -1：不是该进制中的有效数字
```

### 5.3 valueOf、charValue：包装与拆包

```java
Character a = Character.valueOf('a');
char b = a.charValue();

Character c = 'a'; // 自动装箱
char d = c;        // 自动拆箱
```

`new Character('a')` 自 Java 9 起已弃用，优先使用自动装箱或 `valueOf`。

**易错：拆箱 null 会抛出 NullPointerException。**

```java
Character c = null;
// char x = c; // 运行时报错
```

## 6. 字符与数字转换：高频易错点

### 6.1 字符 '7' 变成整数 7

```java
char c = '7';
int digit = c - '0'; // 7
int code = c;        // 55：字符的编码值
```

`'0'`～`'9'` 的编码连续，因此减去 `'0'` 就能得到对应数字。

**使用前提：已确定 c 是 ASCII 范围的 `'0'`～`'9'`。** 不要认为 `isDigit(c)` 为真，就能对所有 Unicode 数字使用 `c - '0'`。

### 6.2 整数 7 变成字符 '7'

```java
int digit = 7; // 前提：0 <= digit <= 9
char c = (char) ('0' + digit); // '7'
```

直接 `(char) 7` 得到的是编码值为 7 的控制字符，不是 `'7'`。

### 6.3 两个 char 相加，默认是数值运算

```java
System.out.println('1' + '2');      // 99：49 + 50
System.out.println("" + '1' + '2'); // 12：字符串拼接
```

## 7. 转义字符速查

这些是 Java 源码中的写法，不是 Character 的方法。

| 写法 | 含义 | 示例 |
| --- | --- | --- |
| `\t` | 制表符 | `"a\tb"` |
| `\b` | 退格控制字符 | `"a\b"` |
| `\n` | 换行符 | `"a\nb"` |
| `\r` | 回车符 | `"a\rb"` |
| `\f` | 换页符 | `"a\fb"` |
| `\'` | 单引号 | `'\''` |
| `\"` | 双引号 | `"他说：\"你好\""` |
| `\\` | 一个反斜杠 | `"C:\\temp"` |

**易错：`\b` 只是一个控制字符，不会直接删除 Java 字符串里前面的字符。** 控制台如何显示它，取决于输出环境。

## 8. 和刷题结合：检查回文

下面适合题目限定 ASCII 输入、忽略非字母数字与大小写的情况：

```java
public boolean isPalindrome(String s) {
    int left = 0;
    int right = s.length() - 1;

    while (left < right) {
        while (left < right
                && !Character.isLetterOrDigit(s.charAt(left))) {
            left++;
        }
        while (left < right
                && !Character.isLetterOrDigit(s.charAt(right))) {
            right--;
        }

        if (Character.toLowerCase(s.charAt(left))
                != Character.toLowerCase(s.charAt(right))) {
            return false;
        }

        left++;
        right--;
    }
    return true;
}
```

注意数字也要保留。比如 `"0P"` 不能只留下 `"P"` 再判回文。

## 9. 进阶提醒：一个 char 不一定是一个完整字符

Java 的 `char` 是一个 16 位 UTF-16 代码单元。有些字符（例如部分 emoji）需要两个 `char`：

```java
String s = "\uD83D\uDE00"; // 😀
System.out.println(s.length()); // 2
System.out.println(s.codePointCount(0, s.length())); // 1
```

要处理完整 Unicode 码点，可使用 Character 判断方法的 `int codePoint` 重载：

```java
for (int i = 0; i < s.length(); ) {
    int cp = s.codePointAt(i);
    boolean letter = Character.isLetter(cp);
    i += Character.charCount(cp);
}
```

本笔记前面的 `char` 示例适合普通英文、数字等场景；完整 Unicode 文本处理需要考虑码点。码点数量也不一定等于用户眼中“一个可见字符”的数量。

## 10. 复习自测

1. `isLetter('中')` 是什么？—— `true`。
2. `isDigit('-')` 是什么？—— `false`，符号要单独处理。
3. `isWhitespace('\t')` 是什么？—— `true`。
4. `!isUpperCase(c)` 一定说明是小写吗？—— 不一定，可能是数字或标点。
5. `toLowerCase(c)` 会直接修改变量 c 吗？—— 不会，需要接住返回值。
6. `'7' - '0'` 和 `(int) '7'` 分别是多少？—— `7` 和 `55`。
7. `isDigit(c)` 为真，就一定能用 `c - '0'` 吗？—— 不一定，可能是非 ASCII 数字。
8. 判断字母或数字用什么？—— `Character.isLetterOrDigit(c)`。

## 参考来源

- 阅读范围：[菜鸟教程：Java Character 类](https://www.runoob.com/java/java-character.html)。本笔记完整覆盖其 8 个方法条目，并使用自行编写的示例解释。
- 方法语义与 Unicode 边界：[Java SE 17 Character 官方 API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Character.html)。
- 字符串补充说明见 [Java String 复习笔记](Java_String_复习笔记.md)。
