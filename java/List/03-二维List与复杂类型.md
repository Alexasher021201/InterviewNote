# Java List 03：二维 List 与复杂类型

> 目标：看懂 LeetCode 中常见的
> `List<List<Integer>>`、`List<int[]>`、`List<TreeNode>`。

## 1. 为什么需要二维 List

很多 LeetCode 题返回的不只是一组数字，而是**多组答案**。

例如：

``` text
[
    [1, 2],
    [1, 3],
    [2, 3]
]
```

这时使用：

``` java
List<List<Integer>> result = new ArrayList<>();
```

外层 List 的每个元素，本身又是一个 `List<Integer>`。

## 2. 创建与添加

``` java
List<List<Integer>> result = new ArrayList<>();

List<Integer> row1 = new ArrayList<>();
row1.add(1);
row1.add(2);

List<Integer> row2 = new ArrayList<>();
row2.add(3);
row2.add(4);

result.add(row1);
result.add(row2);
```

得到：

``` text
[[1, 2], [3, 4]]
```

## 3. 二维 List 读取

假设：

``` text
[
    [1, 2],
    [3, 4],
    [5, 6]
]
```

获取第二行：

``` java
List<Integer> row = result.get(1);
```

获取第二行第二个元素：

``` java
int x = result.get(1).get(1);
```

结果为 `4`。

对应关系：

``` text
二维数组：nums[i][j]

二维 List：list.get(i).get(j)
```

## 4. 二维 List 遍历

普通 for：

``` java
for (int i = 0; i < result.size(); i++) {
    List<Integer> row = result.get(i);

    for (int j = 0; j < row.size(); j++) {
        System.out.println(row.get(j));
    }
}
```

增强 for：

``` java
for (List<Integer> row : result) {
    for (int num : row) {
        System.out.println(num);
    }
}
```

## 5. List\<int\[\]\>

有些题更适合保存多个数组：

``` java
List<int[]> list = new ArrayList<>();

list.add(new int[]{1, 2});
list.add(new int[]{3, 4});
```

获取第一个数组：

``` java
int[] arr = list.get(0);
```

获取第一个数组的第二个元素：

``` java
int x = list.get(0)[1];
```

## 6. 两种二维结构的区别

### List\<List`<Integer>`{=html}\>

``` java
List<List<Integer>> list = new ArrayList<>();
```

访问：

``` java
list.get(i).get(j);
```

因为 `list.get(i)` 得到的是 `List<Integer>`。

### List\<int\[\]\>

``` java
List<int[]> list = new ArrayList<>();
```

访问：

``` java
list.get(i)[j];
```

因为 `list.get(i)` 得到的是 `int[]`。

牢记：

``` text
List<List<Integer>> -> get(i).get(j)

List<int[]>         -> get(i)[j]
```

## 7. List`<TreeNode>`{=html}

List 不只能保存数字。

二叉树题中可以：

``` java
List<TreeNode> nodes = new ArrayList<>();

nodes.add(root);
nodes.add(root.left);
nodes.add(root.right);
```

因此泛型表示的是：

> "这个 List 中每一个元素是什么类型？"

例如：

``` java
List<Integer>
List<String>
List<TreeNode>
List<int[]>
List<List<Integer>>
```

## 8. LeetCode 返回值怎么看

### List`<Integer>`

一组整数：

``` text
[1, 2, 3]
```

### List`<String>`

一组字符串：

``` text
["abc", "def"]
```

### List\<List`<Integer>`\>

多组整数答案：

``` text
[
  [1, 2],
  [3, 4]
]
```

### List\<int\[\]\>

多个整数数组。

### List`<TreeNode>`

多个树节点。

## 9. 高频场景

`List<List<Integer>>` 经常出现在：

- 三数之和
- 子集
- 组合
- 全排列
- 组合总和
- 路径收集

看到题目要求返回"所有方案""所有组合""所有路径"，要对二维 List 保持敏感。
