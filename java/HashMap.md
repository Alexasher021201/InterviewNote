# Java HashMap 基本用法与 LeetCode 重点

## 1. HashMap 是什么？

`HashMap` 是 Java 中非常常用的哈希表实现，用来保存
**Key-Value（键值对）**。

``` text
key -> value
```

例如统计字符出现次数：

``` text
'a' -> 2
'b' -> 3
'c' -> 1
```

适合使用：

``` java
HashMap<Character, Integer> map = new HashMap<>();
```

在 LeetCode 中，看到下面几类需求时，要优先想到 HashMap：

-   统计某个元素出现的次数
-   判断某个元素以前是否出现过
-   根据一个值快速找到另一个值
-   建立两个对象之间的映射关系

------------------------------------------------------------------------

## 2. 创建 HashMap

使用前通常需要：

``` java
import java.util.HashMap;
```

创建：

``` java
HashMap<Character, Integer> map = new HashMap<>();
```

泛型：

``` java
HashMap<Key类型, Value类型>
```

例如：

``` java
HashMap<String, Integer> map = new HashMap<>();
```

表示：

``` text
String -> Integer

"Tom"  -> 90
"Jack" -> 85
```

常见类型：

``` java
HashMap<Integer, Integer> map1 = new HashMap<>();
HashMap<Character, Integer> map2 = new HashMap<>();
HashMap<String, Integer> map3 = new HashMap<>();
```

> 注意：Java 泛型不能直接使用 `int`、`char` 等基本类型，需要使用包装类
> `Integer`、`Character`。

------------------------------------------------------------------------

## 3. put()：添加或修改元素

语法：

``` java
map.put(key, value);
```

例如：

``` java
map.put('a', 1);
map.put('b', 2);
```

此时：

``` text
'a' -> 1
'b' -> 2
```

如果 key 已经存在：

``` java
map.put('a', 5);
```

原来的：

``` text
'a' -> 1
```

会被覆盖为：

``` text
'a' -> 5
```

所以：

> `put()` 既可以添加，也可以修改。

------------------------------------------------------------------------

## 4. get()：根据 key 获取 value

语法：

``` java
map.get(key);
```

例如：

``` java
map.put('a', 5);

int value = map.get('a');
```

得到：

``` text
value = 5
```

### 注意不存在的 key

如果：

``` java
map.get('b');
```

而 `'b'` 不存在，会返回：

``` text
null
```

因此直接写：

``` java
map.get(c) + 1
```

可能因为 `map.get(c)` 是 `null` 而出错。

------------------------------------------------------------------------

## 5. getOrDefault()：不存在时提供默认值【重点】

语法：

``` java
map.getOrDefault(key, defaultValue);
```

例如：

``` java
map.getOrDefault('a', 0);
```

含义：

``` text
如果 a 存在 -> 返回 a 对应的 value
如果 a 不存在 -> 返回 0
```

### LeetCode 高频写法：统计出现次数

``` java
map.put(c, map.getOrDefault(c, 0) + 1);
```

一定要理解这句话：

> 获取 `c` 原来的出现次数，如果不存在就当作 0，然后加 1，再保存回
> HashMap。

例如：

``` java
String s = "aab";
```

执行：

``` java
for (char c : s.toCharArray()) {
    map.put(c, map.getOrDefault(c, 0) + 1);
}
```

过程：

``` text
第一次 a：
不存在 -> 0 + 1
a -> 1

第二次 a：
原来是 1 -> 1 + 1
a -> 2

第一次 b：
不存在 -> 0 + 1
b -> 1
```

最终：

``` text
a -> 2
b -> 1
```

这是字符统计、频率统计类题目的高频模板。

------------------------------------------------------------------------

## 6. containsKey()：判断 key 是否存在【重点】

语法：

``` java
map.containsKey(key);
```

返回：

``` text
存在   -> true
不存在 -> false
```

例如：

``` java
if (map.containsKey('a')) {
    System.out.println("a 存在");
}
```

### LeetCode 中的典型思想

例如两数之和：

``` text
当前数字 = nums[i]
目标 = target

需要寻找：
target - nums[i]
```

可以判断：

``` java
if (map.containsKey(target - nums[i])) {
    // 找到了需要的另一个数字
}
```

所以：

> 当题目问"某个元素之前有没有出现过"时，可以考虑 HashMap / HashSet。

------------------------------------------------------------------------

## 7. remove()：删除键值对

语法：

``` java
map.remove(key);
```

例如：

``` java
map.remove('a');
```

会删除：

``` text
'a' -> value
```

------------------------------------------------------------------------

## 8. size()：获取键值对数量

``` java
int size = map.size();
```

例如：

``` text
a -> 2
b -> 3
c -> 1
```

那么：

``` java
map.size();
```

结果是：

``` text
3
```

注意：

> `size()` 表示 key 的数量，不是所有 value 的总和。

------------------------------------------------------------------------

## 9. isEmpty()：判断 HashMap 是否为空

``` java
map.isEmpty();
```

返回：

``` text
空     -> true
非空   -> false
```

------------------------------------------------------------------------

# 10. 遍历 HashMap

## 10.1 只遍历 value

使用：

``` java
map.values()
```

例如：

``` java
for (int value : map.values()) {
    System.out.println(value);
}
```

如果：

``` text
a -> 2
b -> 3
c -> 5
```

那么遍历得到的 value 就是：

``` text
2
3
5
```

适用于：

> 不关心具体是哪个 key，只关心每个 key 对应的数据。

例如 LeetCode 409
最长回文串中，统计完字符次数之后，只需要关心每种字符出现了多少次：

``` java
for (int count : map.values()) {
    // 判断 count 是奇数还是偶数
}
```

------------------------------------------------------------------------

## 10.2 只遍历 key

使用：

``` java
map.keySet()
```

例如：

``` java
for (char key : map.keySet()) {
    System.out.println(key);
}
```

------------------------------------------------------------------------

## 10.3 同时遍历 key 和 value

使用：

``` java
map.entrySet()
```

例如：

``` java
for (Map.Entry<Character, Integer> entry : map.entrySet()) {
    char key = entry.getKey();
    int value = entry.getValue();

    System.out.println(key + " -> " + value);
}
```

需要：

``` java
import java.util.Map;
```

记忆：

``` text
entry.getKey()   -> key
entry.getValue() -> value
```

------------------------------------------------------------------------

# 11. LeetCode 高频模板

## 模板一：统计字符出现次数

``` java
HashMap<Character, Integer> map = new HashMap<>();

for (char c : s.toCharArray()) {
    map.put(c, map.getOrDefault(c, 0) + 1);
}
```

结果类似：

``` text
"aabbccc"

a -> 2
b -> 2
c -> 3
```

------------------------------------------------------------------------

## 模板二：统计数字出现次数

``` java
HashMap<Integer, Integer> map = new HashMap<>();

for (int num : nums) {
    map.put(num, map.getOrDefault(num, 0) + 1);
}
```

例如：

``` text
nums = [1, 2, 2, 3, 3, 3]
```

最终：

``` text
1 -> 1
2 -> 2
3 -> 3
```

------------------------------------------------------------------------

## 模板三：判断元素是否出现过

``` java
if (map.containsKey(key)) {
    // key 存在
}
```

------------------------------------------------------------------------

## 模板四：保存"值 -\> 下标"

LeetCode 中非常常见：

``` java
HashMap<Integer, Integer> map = new HashMap<>();

for (int i = 0; i < nums.length; i++) {
    map.put(nums[i], i);
}
```

建立：

``` text
数字 -> 下标
```

例如：

``` text
nums = [5, 8, 3]

5 -> 0
8 -> 1
3 -> 2
```

这也是"两数之和"等题目的常见思路。

------------------------------------------------------------------------

# 12. HashMap 常用 API 速查表

  操作         写法                   作用
  ------------ ---------------------- -----------------------
  创建         `new HashMap<>()`      创建 HashMap
  添加/修改    `put(k, v)`            保存 `k -> v`
  查询         `get(k)`               获取 key 对应的 value
  默认值查询   `getOrDefault(k, 0)`   不存在时返回默认值
  判断 key     `containsKey(k)`       判断 key 是否存在
  删除         `remove(k)`            删除键值对
  大小         `size()`               key-value 数量
  判断为空     `isEmpty()`            是否为空
  所有 key     `keySet()`             获取所有 key
  所有 value   `values()`             获取所有 value
  所有键值对   `entrySet()`           获取所有 key-value

------------------------------------------------------------------------

# 13. HashMap 的时间复杂度

HashMap 底层使用哈希机制。

常见操作平均时间复杂度：

``` text
put()         O(1)
get()         O(1)
containsKey() O(1)
remove()      O(1)
```

因此 HashMap 非常适合解决：

> "我需要快速知道某个东西是否存在 / 对应什么值。"

注意：

> `O(1)`
> 是通常所说的平均时间复杂度；极端哈希冲突等情况下不能简单认为永远严格是
> O(1)。

------------------------------------------------------------------------

# 14. HashMap 和 HashSet 怎么区分？

## HashMap

保存：

``` text
key -> value
```

例如：

``` text
字符 -> 出现次数
数字 -> 数组下标
姓名 -> 分数
```

当你需要：

> "这个元素对应什么信息？"

考虑 HashMap。

## HashSet

主要保存：

``` text
元素是否存在
```

例如：

``` text
1
5
8
```

当你只需要：

> "这个元素出现过没有？"

而完全不需要额外 value 时，可以优先考虑 HashSet。

------------------------------------------------------------------------

# 15. 做题时什么时候想到 HashMap？

看到以下关键词，要建立条件反射。

## ① 统计次数

``` text
每个字符出现多少次？
每个数字出现多少次？
```

想到：

``` java
HashMap<元素类型, Integer>
```

经典写法：

``` java
map.put(x, map.getOrDefault(x, 0) + 1);
```

------------------------------------------------------------------------

## ② 判断是否出现过

``` text
之前有没有这个数字？
有没有与当前数字匹配的元素？
```

想到：

``` java
map.containsKey(x)
```

或者：

``` java
HashSet
```

------------------------------------------------------------------------

## ③ 建立映射关系

例如：

``` text
数字 -> 下标
字符 -> 次数
用户 -> 信息
```

想到：

``` java
HashMap
```

------------------------------------------------------------------------

# 16. HashMap 最需要掌握的四个方法

刷 LeetCode 初期，不需要一次背完所有 API。

优先熟练：

``` java
map.put(key, value);

map.get(key);

map.getOrDefault(key, defaultValue);

map.containsKey(key);
```

其中尤其重要的是：

``` java
map.put(c, map.getOrDefault(c, 0) + 1);
```

看到它要能立刻反应：

> 当前元素 `c` 的出现次数 +1。

------------------------------------------------------------------------

# 17. 最终速记

``` text
HashMap
│
├── 本质
│   └── key -> value
│
├── 添加/修改
│   └── put(key, value)
│
├── 查询
│   ├── get(key)
│   └── getOrDefault(key, 默认值)
│
├── 判断存在
│   └── containsKey(key)
│
├── 删除
│   └── remove(key)
│
├── 遍历
│   ├── keySet()   -> key
│   ├── values()   -> value
│   └── entrySet() -> key + value
│
└── LeetCode 高频用途
    ├── 统计出现次数
    ├── 判断元素是否出现
    └── 建立值与信息之间的映射
```

## 一句话总结

> **HashMap 用来建立 `key -> value`
> 的快速映射。刷题时遇到"统计次数、快速查找、判断是否出现、建立映射关系"，都应该考虑哈希表。**

## LeetCode 必背模板

``` java
HashMap<Character, Integer> map = new HashMap<>();

for (char c : s.toCharArray()) {
    map.put(c, map.getOrDefault(c, 0) + 1);
}
```

这段代码的含义：

> **统计字符串中每个字符出现的次数。**
