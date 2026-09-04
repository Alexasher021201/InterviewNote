# Java List 02：遍历、转换、排序与常用操作

> 目标：掌握刷题中围绕 List 的常规数据处理。

## 1. 普通 for 遍历

``` java
for (int i = 0; i < list.size(); i++) {
    int num = list.get(i);
    System.out.println(num);
}
```

优点：同时拥有**下标和元素值**。

## 2. 增强 for

``` java
for (int num : list) {
    System.out.println(num);
}
```

适合只关心元素、不关心下标的场景。

## 3. 遍历时删除的坑

下面的代码容易漏删：

``` java
for (int i = 0; i < list.size(); i++) {
    if (list.get(i) == 2) {
        list.remove(i);
    }
}
```

例如 `[1, 2, 2, 3]`，删除第一个 `2` 后，第二个 `2` 左移，而 `i`
又自增，因此可能被跳过。

一种常见办法是倒序删除：

``` java
for (int i = list.size() - 1; i >= 0; i--) {
    if (list.get(i) == 2) {
        list.remove(i);
    }
}
```

## 4. int\[\] 转 List`<Integer>`

对刷题最直观：

``` java
int[] nums = {1, 2, 3};

List<Integer> list = new ArrayList<>();

for (int num : nums) {
    list.add(num);
}
```

不要把 `int[]` 直接当成 `Integer[]`。

## 5. Integer\[\] 转 List

``` java
Integer[] nums = {1, 2, 3};
List<Integer> list = Arrays.asList(nums);
```

但 `Arrays.asList()` 得到的 List **不能改变长度**：

``` java
list.add(4);    // 错误
list.remove(0); // 错误
```

如果之后需要正常增删：

``` java
List<Integer> list =
        new ArrayList<>(Arrays.asList(1, 2, 3));
```

## 6. List`<Integer>` 转 int\[\]

``` java
int[] nums = new int[list.size()];

for (int i = 0; i < list.size(); i++) {
    nums[i] = list.get(i);
}
```

## 7. List.of

``` java
List<Integer> list = List.of(1, 2, 3);
```

适合快速创建固定内容，但它是不可修改 List：

``` java
list.add(4); // 错误
```

刷题中如果需要继续增删，使用：

``` java
List<Integer> list = new ArrayList<>();
```

## 8. 升序排序

``` java
Collections.sort(list);
```

也可以：

``` java
list.sort(Integer::compare);
```

例如：

``` text
[3, 1, 2] -> [1, 2, 3]
```

## 9. 降序排序

推荐：

``` java
list.sort((a, b) -> Integer.compare(b, a));
```

不要过度依赖：

``` java
(a, b) -> b - a
```

因为极端情况下减法可能整数溢出。

## 10. 反转

``` java
Collections.reverse(list);
```

例如：

``` text
[1, 2, 3, 4]
->
[4, 3, 2, 1]
```

## 11. List`<String>`

``` java
List<String> words = new ArrayList<>();
words.add("abc");
words.add("leetcode");

String s = words.get(0);
char c = words.get(0).charAt(1); // 'b'
```

## 12. 常用速查

```
  需求          写法
  ------------- --------------------------------------------
  普通遍历      `for (int i=0; i<list.size(); i++)`
  增强 for      `for (int x : list)`
  升序          `Collections.sort(list)`
  降序          `list.sort((a,b) -> Integer.compare(b,a))`
  反转          `Collections.reverse(list)`
  数组转 List   循环 `add()`
  List 转数组   创建数组后循环赋值
```