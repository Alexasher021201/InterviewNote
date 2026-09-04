# Java List 04：引用、复制、底层原理与复杂度

> 目标：理解 `new ArrayList<>(list)` 为什么重要，以及 ArrayList
> 为什么有些操作快、有些操作慢。

## 1. List 变量保存的是引用

``` java
List<Integer> a = new ArrayList<>();
a.add(1);
a.add(2);

List<Integer> b = a;
```

`b = a` **没有创建一个新的 List**。

可以理解成：

``` text
a -----\
        -> [1, 2]
b -----/
```

因此：

``` java
b.add(3);
```

之后：

``` text
a = [1, 2, 3]
b = [1, 2, 3]
```

因为两者指向同一个对象。

## 2. 真正创建一个新的 List

``` java
List<Integer> b = new ArrayList<>(a);
```

可以理解成：

``` text
a -> [1, 2]

b -> [1, 2]
```

此时：

``` java
b.add(3);
```

结果：

``` text
a = [1, 2]
b = [1, 2, 3]
```

## 3. 为什么回溯必须复制

回溯中经常有：

``` java
List<List<Integer>> result = new ArrayList<>();
List<Integer> path = new ArrayList<>();
```

如果写：

``` java
result.add(path);
```

保存进去的是 `path` 的引用。

后续继续：

``` java
path.add(...);
path.remove(...);
```

之前的结果也会受到影响。

因此保存当前路径时通常写：

``` java
result.add(new ArrayList<>(path));
```

含义：

> 把 path 当前的元素复制到一个新的 ArrayList，再把新对象放入 result。

这是 LeetCode 回溯中的核心知识。

## 4. 这是浅拷贝

``` java
new ArrayList<>(list)
```

会创建新的外层 List，但里面的元素对象本身不会递归深复制。

对于：

``` java
List<Integer>
```

对刷题通常足够。

如果 List 中保存的是可变对象，例如：

``` java
List<int[]>
List<TreeNode>
```

就要意识到内部对象仍然可能共享。

## 5. ArrayList 的底层

对刷题可以粗略理解为：

``` text
ArrayList
   |
   v
Object[]
```

也就是一个可以自动扩容的动态数组。

当容量不足时，会申请更大的内部数组并搬运数据。

因此它既具有数组的快速随机访问，又能动态增长。

## 6. 为什么 get 很快

``` java
list.get(i)
```

可以根据下标直接定位元素，因此通常：

``` text
O(1)
```

`set(i, x)` 同样通常为 `O(1)`。

## 7. 为什么中间删除较慢

例如：

``` text
[10, 20, 30, 40]
```

删除 `20` 后：

``` text
[10, 30, 40]
```

后面的元素需要向前移动。

因此：

``` java
list.remove(i);
```

通常为：

``` text
O(n)
```

中间插入同理。

## 8. 高频复杂度

  操作              ArrayList 时间复杂度
  --------------- ----------------------
  `get(i)`                        `O(1)`
  `set(i,x)`                      `O(1)`
  尾部 `add(x)`              平均 `O(1)`
  `add(i,x)`                      `O(n)`
  `remove(i)`                     `O(n)`
  `contains(x)`                   `O(n)`
  `indexOf(x)`                    `O(n)`

## 9. ArrayList 与 LinkedList

`ArrayList` 底层接近动态数组。

`LinkedList` 底层是链式结构。

对刷 LeetCode：

> 不知道该选哪个时，默认 `ArrayList`。

尤其需要频繁：

``` java
list.get(i)
```

时，`ArrayList` 通常更加自然。

## 10. Integer 比较

`List<Integer>` 保存的是 `Integer` 对象。

比较对象值时，更稳妥的是：

``` java
a.equals(b)
```

或：

``` java
Objects.equals(a, b)
```

不要把对象引用比较 `==` 和数值比较混为一谈。

由于自动拆箱，下面这种刷题代码通常很自然：

``` java
int x = list.get(i);

if (x == target) {
    // ...
}
```

此时比较的是基本类型 `int`。

## 11. 核心结论

必须真正理解这两行的区别：

``` java
List<Integer> b = a;
```

这是**共享同一个 List**。

``` java
List<Integer> b = new ArrayList<>(a);
```

这是**创建一个新的 List，并复制当前元素**。
