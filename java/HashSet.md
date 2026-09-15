可以。下面我把它整理成一份更适合你**后续刷 LeetCode 复习**的版本，不追求 API 大全，重点围绕你现在最需要的：**HashSet 是什么、什么时候用、核心语法、和 List 的区别、链表判环、常见坑和刷题思维**。

# Java HashSet｜LeetCode 刷题复习笔记

> 核心定位：`HashSet` 最擅长解决  
>
> **“这个东西以前出现过吗？”**

---

# 1. HashSet 是什么？

创建：

```java
Set<Integer> set = new HashSet<>();
```

添加：

```java
set.add(10);
set.add(20);
set.add(30);
```

可以理解成：hash集合

```text
set = {10, 20, 30}
```

HashSet 最重要的特点：

- **不允许重复元素**
- **没有下标**
- 不应依赖元素的遍历顺序
- `add()`、`contains()`、`remove()` 平均为 `O(1)`
- 非常适合**去重、判断是否出现过、记录 visited**

例如：

```java
set.add(10);
set.add(10);
set.add(10);
```

最终仍然只有一个 `10`。

---

# 2. HashSet 和 ArrayList 的区别

这是最需要建立的概念。

| 特点 | ArrayList | HashSet |
|---|---|---|
| 允许重复 | ✅ | ❌ |
| 有下标 | ✅ | ❌ |
| `get(i)` | ✅ | ❌ |
| `contains(x)` | O(n) | 平均 O(1) |
| 主要用途 | 保存一串数据 | 去重 / 判断存在 |
| 常见思维 | “第几个元素？” | “出现过没有？” |

所以：

```text
需要保存：

第0个
第1个
第2个
...

→ ArrayList
```

而：

```text
需要判断：

x 出现过吗？
这个节点访问过吗？
这个字符重复了吗？

→ HashSet
```

---

# 3. HashSet 基础语法

## 创建

整数：

```java
Set<Integer> set = new HashSet<>();
```

字符串：

```java
Set<String> set = new HashSet<>();
```

字符：

```java
Set<Character> set = new HashSet<>();
```

链表节点：

```java
Set<ListNode> set = new HashSet<>();
```

注意泛型不能写基本类型：

```java
Set<int> set;       // ❌

Set<Integer> set;   // ✅
```

---

# 4. 核心 API

## add：添加

```java
set.add(x);
```

例如：

```java
Set<Integer> set = new HashSet<>();

set.add(10);
set.add(20);
```

---

## contains：判断是否存在

```java
set.contains(x);
```

例如：

```java
if (set.contains(10)) {
    System.out.println("出现过");
}
```

这是 HashSet **对刷题最重要的 API**。

---

## remove：删除

```java
set.remove(x);
```

例如：

```java
set.remove(10);
```

---

## size：元素数量

```java
int size = set.size();
```

注意不是：

```java
set.length; // ❌
```

---

## isEmpty：判断为空

```java
if (set.isEmpty()) {

}
```

---

## clear：清空

```java
set.clear();
```

---

# 5. add() 本身有返回值

这一点刷题很好用。

```java
set.add(x)
```

返回 `boolean`。

如果 `x` 原来不存在：

```java
set.add(x)
```

返回：

```text
true
```

如果已经存在：

```java
set.add(x)
```

返回：

```text
false
```

例如：

```java
Set<Integer> set = new HashSet<>();

set.add(10); // true

set.add(10); // false
```

因此：

```java
if (set.contains(x)) {
    return true;
}

set.add(x);
```

可以简化成：

```java
if (!set.add(x)) {
    return true;
}
```

刚开始学习时建议先写 `contains + add`，逻辑更加直观。

---

# 6. HashSet 没有下标

错误：

```java
set.get(0); // ❌
```

错误：

```java
set[0]; // ❌
```

HashSet 不关心：

```text
第0个
第1个
第2个
```

它关心：

```text
这个元素存在吗？
```

所以核心操作是：

```java
set.contains(x);
```

而不是：

```java
set.get(i);
```

---

# 7. HashSet 的遍历

因为没有下标，通常使用增强 for：

```java
Set<Integer> set = new HashSet<>();

set.add(10);
set.add(20);
set.add(30);

for (int num : set) {
    System.out.println(num);
}
```

不要依赖 HashSet 的遍历顺序。

也就是说：

```java
set.add(10);
set.add(20);
set.add(30);
```

不代表遍历一定得到：

```text
10 → 20 → 30
```

---

# 8. HashSet 最经典用途①：判断重复

例如：

```text
nums = [1, 2, 3, 1]
```

从左往右：

```text
set = {}
```

遇到 `1`：

```text
没出现过
↓
加入

{1}
```

遇到 `2`：

```text
没出现过
↓
加入

{1,2}
```

遇到 `3`：

```text
没出现过
↓
加入

{1,2,3}
```

再次遇到 `1`：

```text
set.contains(1)

→ true
```

说明存在重复。

代码：

```java
public boolean containsDuplicate(int[] nums) {

    Set<Integer> set = new HashSet<>();

    for (int num : nums) {

        if (set.contains(num)) {
            return true;
        }

        set.add(num);
    }

    return false;
}
```

典型题：

> LeetCode 217：存在重复元素

---

# 9. HashSet 最经典用途②：去重

例如：

```java
int[] nums = {1, 2, 2, 3, 3, 3};

Set<Integer> set = new HashSet<>();

for (int num : nums) {
    set.add(num);
}
```

最终 Set 中只有：

```text
1
2
3
```

所以看到：

```text
去重
不同元素
不重复元素数量
```

可以考虑：

```java
HashSet
```

---

# 10. HashSet 最经典用途③：visited

以后 DFS、BFS、图、链表里经常看到：

```java
Set<Node> visited = new HashSet<>();
```

`visited`：

```text
已经访问过的
```

核心逻辑：

```java
if (visited.contains(node)) {
    // 来过了
}

visited.add(node);
```

可以理解成：

```text
来到一个节点
    ↓
以前来过吗？
    ↓
 ┌──┴──┐
 是     否
 ↓       ↓
重复    加入 visited
```

这和二维数组 DFS 中：

```java
boolean[][] visited;
```

思想完全相同。

---

# 11. LeetCode 141：环形链表

这是目前非常重要的 HashSet 实战。

普通链表：

```text
1 → 2 → 3 → 4 → null
```

最终一定走到：

```text
null
```

但如果存在环：

```text
1 → 2 → 3 → 4
        ↑       |
        └───────┘
```

遍历会：

```text
1
↓
2
↓
3
↓
4
↓
3
↓
4
↓
3
...
```

永远不会到 `null`。

---

# 12. 环形链表为什么适合 HashSet？

核心问题可以转换成：

> **当前这个节点，我以前有没有访问过？**

所以：

```java
Set<ListNode> visited = new HashSet<>();
```

遍历链表：

```java
ListNode current = head;

while (current != null) {

    if (visited.contains(current)) {
        return true;
    }

    visited.add(current);

    current = current.next;
}
```

如果某个节点第二次出现：

```text
这个节点以前访问过
        ↓
链表绕回来了
        ↓
存在环
```

---

# 13. 环形链表为什么是 Set<ListNode>？

这是一个重点难点。

正确：

```java
Set<ListNode> visited = new HashSet<>();
```

而不是：

```java
Set<Integer> visited = new HashSet<>();
```

为什么不能保存：

```java
node.val
```

？

因为链表允许不同节点拥有相同的值。

例如：

```text
      节点A        节点B
        ↓            ↓

1 →    [2] → 3 →    [2] → null
```

这里两个节点：

```text
val 都是 2
```

但是：

```text
节点A != 节点B
```

链表也没有环。

如果保存：

```java
visited.add(node.val);
```

第二次遇到 `2` 就会错误判断：

```text
有环
```

---

# 14. 环真正判断的是什么？

不是：

> “这个值以前出现过吗？”

而是：

> **“这个节点对象本身以前出现过吗？”**

例如：

```text
       ┌──────────────┐
       ↓              |
1 → 节点A → 节点B → 节点C
```

再次回到：

```text
节点A
```

说明：

```text
同一个节点对象
第二次访问
```

因此：

```java
visited.contains(current)
```

判断的是节点对象。

这也是理解链表时非常重要的：

> **链表操作的核心很多时候是节点引用，而不仅仅是 `val`。**

---

# 15. 环形链表 HashSet 完整模板

推荐现阶段使用这个版本：

```java
public boolean hasCycle(ListNode head) {

    Set<ListNode> visited = new HashSet<>();

    ListNode current = head;

    while (current != null) {

        if (visited.contains(current)) {
            return true;
        }

        visited.add(current);

        current = current.next;
    }

    return false;
}
```

思维过程：

```text
current
   ↓
这个节点访问过吗？
   ↓
 ┌─┴─┐
是   否
↓     ↓
有环  加入Set
      ↓
 current.next
```

---

# 16. 环形链表简化写法

因为：

```java
set.add(x)
```

第一次添加：

```text
true
```

重复添加：

```text
false
```

所以可以：

```java
public boolean hasCycle(ListNode head) {

    Set<ListNode> visited = new HashSet<>();

    while (head != null) {

        if (!visited.add(head)) {
            return true;
        }

        head = head.next;
    }

    return false;
}
```

现阶段理解第一版更重要，不必追求代码短。

---

# 17. HashSet 解环形链表的复杂度

假设有 `n` 个节点。

每个节点最多访问一次：

```text
时间复杂度：

O(n)
```

HashSet 最坏需要保存所有访问过的节点：

```text
空间复杂度：

O(n)
```

所以：

```text
HashSet 判环

时间：O(n)
空间：O(n)
```

---

# 18. 环形链表还有更优解：快慢指针

HashSet 不是 141 的最优空间解法。

还可以：

```java
ListNode slow = head;
ListNode fast = head;
```

每次：

```java
slow = slow.next;

fast = fast.next.next;
```

如果存在环：

```text
fast 最终会追上 slow
```

类似两个运动员在环形跑道：

```text
slow：一次走1步
fast：一次走2步
```

有环时最终相遇。

复杂度：

```text
时间：O(n)
空间：O(1)
```

学习顺序建议：

```text
HashSet 判环
    ↓
理解“节点是否访问过”
    ↓
快慢指针判环
    ↓
理解空间优化
```

---

# 19. HashSet 的时间复杂度

对刷 LeetCode 重点记：

| 操作 | 平均时间复杂度 |
|---|---:|
| `add(x)` | O(1) |
| `contains(x)` | O(1) |
| `remove(x)` | O(1) |

对比：

```java
ArrayList.contains(x)
```

通常：

```text
O(n)
```

所以如果题目需要**频繁判断某个东西是否存在**：

```text
ArrayList
    ↓
可能每次都要遍历

HashSet
    ↓
平均 O(1)
```

HashSet 往往更加合适。

---

# 20. 常见错误

## 错误① HashSet<int>

```java
HashSet<int> set; // ❌
```

应该：

```java
HashSet<Integer> set; // ✅
```

---

## 错误② 把 HashSet 当 List

```java
set.get(0); // ❌

set[0]; // ❌
```

HashSet 没有下标。

---

## 错误③ contains 后忘记 add

错误：

```java
if (set.contains(x)) {
    return true;
}

// 忘了记录 x
```

应该：

```java
if (set.contains(x)) {
    return true;
}

set.add(x);
```

逻辑：

```text
先判断以前有没有
↓
没有
↓
记录“现在见过了”
```

---

## 错误④ 环形链表保存 val

错误：

```java
Set<Integer> visited = new HashSet<>();

visited.add(node.val);
```

因为：

```text
值相同 ≠ 节点相同
```

正确：

```java
Set<ListNode> visited = new HashSet<>();

visited.add(node);
```

---

# 21. HashSet 高频题型

以后看到这些关键词，可以考虑 HashSet：

```text
重复
去重
是否出现过
是否访问过
不同元素
交集
环
无重复字符
```

建立条件反射：

```text
题目问：

“以前有没有出现过？”

        ↓

    HashSet
```

---

# 22. HashSet 与其他知识的联系

## 和 List

```text
List
↓
保存一串有顺序的数据
↓
list.get(i)
```

HashSet：

```text
HashSet
↓
判断某东西是否存在
↓
set.contains(x)
```

---

## 和链表

```java
Set<ListNode> visited;
```

记录：

```text
哪些节点对象访问过
```

---

## 和二维数组 DFS

以前：

```java
boolean[][] visited;
```

现在：

```java
Set<Node> visited;
```

其实都是：

> **记录已经访问过的状态，防止重复访问。**

---

## 和图

以后图的 DFS/BFS 中经常出现：

```java
Set<Integer> visited = new HashSet<>();
```

或者：

```java
Set<Node> visited = new HashSet<>();
```

所以环形链表其实是在提前训练以后图算法里的 `visited` 思维。

---

# 23. HashSet 刷题速查表

```java
// 创建
Set<Integer> set = new HashSet<>();


// 添加
set.add(x);


// 判断存在
set.contains(x);


// 删除
set.remove(x);


// 元素数量
set.size();


// 是否为空
set.isEmpty();


// 清空
set.clear();


// 遍历
for (int x : set) {

}
```

链表：

```java
Set<ListNode> visited = new HashSet<>();

visited.add(node);

visited.contains(node);
```

---

# 24. 现阶段必须掌握的 5 件事

### ① HashSet 不允许重复

```java
set.add(1);
set.add(1);
```

Set 中仍然只有一个 `1`。

### ② HashSet 没有下标

没有：

```java
set.get(i);
```

核心是：

```java
set.contains(x);
```

### ③ contains 平均 O(1)

所以特别适合：

```text
判断是否出现过
```

### ④ HashSet 可以保存对象

例如：

```java
Set<ListNode>
```

不只能保存 Integer。

### ⑤ 环形链表保存节点，不保存 val

牢记：

```text
值相同

≠

节点相同
```

LeetCode 141 判断的是：

```text
同一个节点有没有再次出现
```

---

# 25. 推荐练习顺序

```text
LeetCode 217
存在重复元素
│
│ 练：
│ contains + add
↓
LeetCode 349
两个数组的交集
│
│ 练：
│ Set 去重 + contains
↓
LeetCode 141
环形链表
│
│ 练：
│ Set<ListNode>
│ visited 思维
↓
LeetCode 141
快慢指针版本
```

---

# 最终记忆图

```text
                  HashSet
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
       去重       判断存在       visited
        │            │            │
        ↓            ↓            ↓
    不允许重复    contains()   是否访问过
                     │            │
                     │            ↓
                     │       链表 / DFS / BFS
                     │
                     ↓
                 平均 O(1)
```

最后只记一句：

> **List 更关心“第几个、按什么顺序保存”；HashSet 更关心“这个东西以前出现过没有”。**

环形链表再加一句：

> **判断环不是判断 `node.val` 是否重复，而是判断同一个 `ListNode` 节点对象是否再次被访问。**

后续复习这份时，建议优先看 **第 2、5、12～18、20、24 节**。这些基本覆盖了你目前使用 HashSet 刷题最需要掌握的部分。