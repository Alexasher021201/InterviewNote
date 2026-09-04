# Java List 06：LeetCode 速查表与易错点

> 这份文件用于刷题时快速查语法。忘记 API 时优先打开这一份。

## 1. 创建

``` java
List<Integer> list = new ArrayList<>();

List<String> words = new ArrayList<>();

List<TreeNode> nodes = new ArrayList<>();

List<List<Integer>> result = new ArrayList<>();

List<int[]> intervals = new ArrayList<>();
```

## 2. 一维 List 核心 API

``` java
list.add(x);                 // 尾部添加
list.add(i, x);              // 在下标 i 插入

list.get(i);                 // 获取

list.set(i, x);              // 修改

list.size();                 // 长度

list.remove(i);              // 删除下标 i

list.remove(list.size()-1);  // 删除最后一个

list.contains(x);            // 是否存在

list.indexOf(x);             // 第一次出现的位置

list.isEmpty();              // 是否为空

list.clear();                // 清空
```

## 3. 数组 / String / List 对照

  类型                   长度            获取
  ---------------------- --------------- ---------------
  `int[] nums`           `nums.length`   `nums[i]`
  `String s`             `s.length()`    `s.charAt(i)`
  `List<Integer> list`   `list.size()`   `list.get(i)`

修改：

``` java
nums[i] = x;

list.set(i, x);
```

## 4. 遍历

普通 for：

``` java
for (int i = 0; i < list.size(); i++) {
    int x = list.get(i);
}
```

增强 for：

``` java
for (int x : list) {
    // ...
}
```

## 5. 排序

升序：

``` java
Collections.sort(list);
```

降序：

``` java
list.sort((a, b) -> Integer.compare(b, a));
```

反转：

``` java
Collections.reverse(list);
```

## 6. 二维 List

创建：

``` java
List<List<Integer>> result = new ArrayList<>();
```

添加一组：

``` java
List<Integer> row = new ArrayList<>();
row.add(1);
row.add(2);

result.add(row);
```

读取：

``` java
result.get(i).get(j);
```

遍历：

``` java
for (List<Integer> row : result) {
    for (int x : row) {
        // ...
    }
}
```

## 7. 复制

共享同一个对象：

``` java
List<Integer> b = a;
```

复制成新的 List：

``` java
List<Integer> b = new ArrayList<>(a);
```

回溯保存答案：

``` java
result.add(new ArrayList<>(path));
```

## 8. 回溯模板

``` java
path.add(nums[i]);

dfs(...);

path.remove(path.size() - 1);
```

保存：

``` java
result.add(new ArrayList<>(path));
```

## 9. 高频错误 1：List`<int>`{=html}

错误：

``` java
List<int> list = new ArrayList<>();
```

正确：

``` java
List<Integer> list = new ArrayList<>();
```

## 10. 高频错误 2：把 List 当数组

错误：

``` java
list.length
list[i]
list[i] = x;
```

正确：

``` java
list.size()
list.get(i)
list.set(i, x)
```

## 11. 高频错误 3：空 List 直接 set

错误：

``` java
List<Integer> list = new ArrayList<>();
list.set(0, 10);
```

正确：

``` java
list.add(10);
```

`set` 是修改，不是添加。

## 12. 高频错误 4：remove(Integer)

假设：

``` java
List<Integer> list = new ArrayList<>();
```

``` java
list.remove(1);
```

表示：

> 删除下标 1。

如果要删除数值 `1`：

``` java
list.remove(Integer.valueOf(1));
```

## 13. 高频错误 5：回溯直接保存 path

容易出错：

``` java
result.add(path);
```

通常应该：

``` java
result.add(new ArrayList<>(path));
```

## 14. 高频错误 6：遍历时正序删除

容易漏元素：

``` java
for (int i = 0; i < list.size(); i++) {
    if (...) {
        list.remove(i);
    }
}
```

很多按条件删除的场景可以考虑倒序：

``` java
for (int i = list.size() - 1; i >= 0; i--) {
    if (...) {
        list.remove(i);
    }
}
```

## 15. 高频错误 7：Arrays.asList 后 add

``` java
List<Integer> list = Arrays.asList(1, 2, 3);

list.add(4); // 错误
```

需要可变长度：

``` java
List<Integer> list =
        new ArrayList<>(Arrays.asList(1, 2, 3));
```

## 16. 高频错误 8：List.of 后修改

``` java
List<Integer> list = List.of(1, 2, 3);
list.add(4); // 错误
```

`List.of()` 创建的是不可修改 List。

## 17. 高频复杂度

  API               ArrayList
  --------------- -----------
  `get(i)`               O(1)
  `set(i,x)`             O(1)
  尾部 `add(x)`     平均 O(1)
  `remove(i)`            O(n)
  `contains(x)`          O(n)
  `indexOf(x)`           O(n)

## 18. 刷题时最值得背的 10 行

``` java
List<Integer> list = new ArrayList<>();

list.add(x);

int x = list.get(i);

list.set(i, x);

int n = list.size();

list.remove(list.size() - 1);

List<List<Integer>> result = new ArrayList<>();

result.add(new ArrayList<>(list));

Collections.sort(list);

for (int x : list) {
    // ...
}
```

如果这些已经形成肌肉记忆，绝大多数 LeetCode 题里遇到 List
时就不会因为语法卡住。
