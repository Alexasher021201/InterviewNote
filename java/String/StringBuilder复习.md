# StringBuilder 复习

## 1. 为什么需要它？

`String` 是**不可变对象**。所谓“修改”字符串，实际上是创建一个新的 `String`：

```java
String s = "hello";
s += " world"; // 创建新字符串，不是原地修改
```

少量拼接没有问题；但在循环中频繁拼接时，会不断创建中间对象，效率较低。

`StringBuilder` 是**可变字符序列**，可在原对象上追加、删除、修改和反转字符，因此适合频繁修改文本。

---

## 2. 什么时候用？

- 循环中拼接字符串
- 构造较长的输出内容
- 需要反转、插入、删除或按下标修改字符
- 算法题中的字符串处理

普通字符串常量、很少次数的拼接，直接使用 `String` 即可。

---

## 3. 基本写法

```java
StringBuilder sb = new StringBuilder();
sb.append("hello");
sb.append(" world");

String result = sb.toString(); // 转成 String
```

也可以由已有字符串创建：

```java
StringBuilder sb = new StringBuilder("abc");
```

---

## 4. 高频方法

| 目的 | 方法 | 示例 |
|---|---|---|
| 末尾追加 | `append(x)` | `sb.append("Java")` |
| 获取字符 | `charAt(i)` | `sb.charAt(0)` |
| 修改字符 | `setCharAt(i, c)` | `sb.setCharAt(0, 'A')` |
| 插入内容 | `insert(i, x)` | `sb.insert(1, "X")` |
| 删除区间 | `delete(l, r)` | 删除 `[l, r)` |
| 删除一个字符 | `deleteCharAt(i)` | `sb.deleteCharAt(0)` |
| 反转 | `reverse()` | `sb.reverse()` |
| 当前长度 | `length()` | `sb.length()` |
| 转回字符串 | `toString()` | `sb.toString()` |

注意：`delete(l, r)` 删除左闭右开区间 `[l, r)`。

---

## 5. 链式调用

大部分修改方法返回当前 `StringBuilder` 对象：

```java
String text = new StringBuilder()
        .append("姓名：")
        .append("张三")
        .append("，年龄：")
        .append(18)
        .toString();
```

---

## 6. 经典：反转字符串

```java
String s = "abcde";
String reversed = new StringBuilder(s).reverse().toString();

System.out.println(reversed); // edcba
```

`reverse()` 会修改 `StringBuilder` 本身；如果最终需要 `String`，再调用 `toString()`。

---

## 7. String / StringBuilder / StringBuffer

| 类型 | 是否可变 | 适用场景 |
|---|---|---|
| `String` | 否 | 普通文本、少量拼接 |
| `StringBuilder` | 是 | 单线程下频繁修改字符串，最常用 |
| `StringBuffer` | 是 | 多线程共享同一对象且需要同步时 |

刷题和绝大多数日常开发中，频繁拼接文本优先使用 `StringBuilder`。

## 一句话记忆

**String 不可变；要频繁改字符串，用 StringBuilder；完成后用 `toString()` 取结果。**
