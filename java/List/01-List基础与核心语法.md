# Java List 01：基础与核心语法

> 目标：对刷 LeetCode 来说，先掌握 List
> 最常见的创建、读取、修改、添加和删除。

## 1. List 是什么

`List` 是 Java
集合框架中的接口，用于保存**有顺序、允许重复**的一组数据。

可以先把它理解成"长度可以动态变化的数组"。

``` java
List<Integer> list = new ArrayList<>();
list.add(10);
list.add(20);
list.add(30);
// [10, 20, 30]
```

数组和 List 最直观的区别：

```
  操作       数组            List
  ---------- --------------- ---------------------
  创建       `new int[n]`    `new ArrayList<>()`
  长度       `nums.length`   `list.size()`
  读取       `nums[i]`       `list.get(i)`
  修改       `nums[i] = x`   `list.set(i, x)`
  尾部添加   不方便          `list.add(x)`
  删除       不方便          `list.remove(i)`
```

## 2. List 与 ArrayList

`List` 是接口，`ArrayList` 是它最常用的实现类：

``` java
List<Integer> list = new ArrayList<>();
```

对刷题而言，如果没有特殊理由，优先使用 `ArrayList`。

``` text
          List
         /    \
 ArrayList   LinkedList
```

## 3. 泛型与包装类

不能写：

``` java
List<int> list = new ArrayList<>(); // 错误
```

应该写：

``` java
List<Integer> list = new ArrayList<>();
```

常见对应关系：
```
  基本类型    包装类
  ----------- -------------
  `int`       `Integer`
  `long`      `Long`
  `double`    `Double`
  `char`      `Character`
  `boolean`   `Boolean`
```

例如：

``` java
List<Integer> nums = new ArrayList<>();
List<String> words = new ArrayList<>();
List<Character> chars = new ArrayList<>();
```

## 4. add：添加

尾部添加：

``` java
list.add(10);
list.add(20);
```

指定位置插入：

``` java
list.add(1, 100);
```

如果原来是 `[10, 20, 30]`，结果是：

``` text
[10, 100, 20, 30]
```

## 5. get：读取

数组使用：

``` java
nums[i]
```

List 使用：

``` java
list.get(i)
```

例如：

``` java
int x = list.get(0);
```

## 6. size：长度

``` java
int n = list.size();
```

牢记：

``` text
数组：   nums.length
String： s.length()
List：   list.size()
```

## 7. set：修改

``` java
list.set(index, value);
```

例如：

``` java
List<Integer> list = new ArrayList<>();
list.add(10);
list.add(20);
list.set(1, 100);
// [10, 100]
```

`set` 只能修改已经存在的位置：

``` java
List<Integer> list = new ArrayList<>();
list.set(0, 10); // IndexOutOfBoundsException
```

空 List 应该先 `add()`。

## 8. remove：删除

按下标删除：

``` java
list.remove(1);
```

删除最后一个元素：

``` java
list.remove(list.size() - 1);
```

### Integer 的特殊坑

对于 `List<Integer>`：

``` java
list.remove(1);
```

表示删除**下标 1**。

如果要删除数值 `1`：

``` java
list.remove(Integer.valueOf(1));
```

## 9. contains / indexOf

判断是否包含：

``` java
if (list.contains(10)) {
    // ...
}
```

寻找第一次出现的位置：

``` java
int index = list.indexOf(10);
```

不存在时返回 `-1`。

注意 `ArrayList.contains()` 和 `indexOf()` 都需要线性查找，通常为
`O(n)`。

## 10. isEmpty / clear

判断是否为空：

``` java
if (list.isEmpty()) {
    // ...
}
```

清空：

``` java
list.clear();
```

## 11. 自动装箱与拆箱

``` java
List<Integer> list = new ArrayList<>();
list.add(10);
```

这里 Java 自动把 `int` 转成 `Integer`，称为**自动装箱**。

``` java
int x = list.get(0);
```

这里 Java 自动把 `Integer` 转成 `int`，称为**自动拆箱**。

## 12. LeetCode 最低限度必须熟练

``` java
List<Integer> list = new ArrayList<>();

list.add(x);
list.get(i);
list.set(i, x);
list.size();
list.remove(i);
list.remove(list.size() - 1);
list.contains(x);
list.isEmpty();
```

一句话记忆：

> 数组主要通过 `[]` 操作；List 主要通过方法操作。
