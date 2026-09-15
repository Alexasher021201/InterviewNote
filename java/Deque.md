可以。你现在学 `Queue` 的目标非常明确：**为了写二叉树层序遍历 BFS**。所以不需要先学一大堆 Queue API，我们围绕 LeetCode 高频用法来掌握。

你可以先记一句：

> **Queue（队列）= 先进先出 FIFO。**

# Java Queue｜二叉树层序遍历入门

## 1. Queue 是什么？

想象排队买饭：

```text
进入队列：

A → B → C

A 最先来
↓
A 最先离开
```

所以 Queue 的特点：

```text
First In First Out
       ↓
     FIFO
       ↓
    先进先出
```

例如：

```text
入队顺序：

1 → 2 → 3

出队顺序：

1 → 2 → 3
```

这和栈 `Stack` 正好不同：

```text
Queue：先进先出
Stack： 后进先出
```

---

# 2. Java 中怎么创建 Queue？

对刷 LeetCode，推荐先这样写：

```java
Queue<TreeNode> queue = new LinkedList<>(); 
```

需要：

```java
import java.util.Queue;
import java.util.LinkedList;
```

LeetCode 通常已经处理好 `java.util.*`。

也经常推荐使用：

```java
Queue<TreeNode> queue = new ArrayDeque<>();
```

对于普通 BFS，`ArrayDeque` 通常是更好的默认选择。

你现阶段可以统一写：

```java
Queue<TreeNode> queue = new ArrayDeque<>();
```

---

# 3. 为什么是 `Queue<TreeNode>`？

和你之前学：

```java
List<Integer>
Set<ListNode>
```

完全是一个泛型思想。

```java
Queue<TreeNode>
```

表示：

> 这个 Queue 里面保存的是 `TreeNode`。

例如有：

```text
        3
       / \
      9   20
```

可以：

```java
queue.offer(root);
queue.offer(root.left);
queue.offer(root.right);
```

队列里就是：

```text
队头                    队尾
 ↓                       ↓

[3] → [9] → [20]
```

---

# 4. Queue 现阶段只需要掌握 4 个 API

对刷二叉树 BFS，你先掌握：

```java
queue.offer(x);   // 入队

queue.poll();     // 出队，并返回队头

queue.peek();     // 看队头，但不删除

queue.size();     // 当前元素数量
```

再加：

```java
queue.isEmpty();
```

判断是否为空。

其中最重要的是：

```text
offer → 进去
poll  → 出来
```

---

# 5. offer()：入队

例如：

```java
Queue<Integer> queue = new ArrayDeque<>();

queue.offer(10);
queue.offer(20);
queue.offer(30);
```

队列：

```text
队头                    队尾
 ↓                       ↓

10 → 20 → 30
```

最先进去的是：

```text
10
```

所以它最先出来。

---

# 6. poll()：出队

现在：

```text
10 → 20 → 30
```

执行：

```java
int num = queue.poll();
```

得到：

```text
num = 10
```

同时队列变成：

```text
20 → 30
```

再：

```java
queue.poll();
```

得到：

```text
20
```

队列：

```text
30
```

所以：

> `poll()` 不仅获取队头，还会把队头从 Queue 中删除。

---

# 7. peek()：只看，不删除

假设：

```text
10 → 20 → 30
```

执行：

```java
int num = queue.peek();
```

得到：

```text
num = 10
```

但是队列仍然：

```text
10 → 20 → 30
```

所以：

```text
poll()

拿出来 + 删除
```

而：

```text
peek()

看一眼 + 不删除
```

---

# 8. isEmpty()

判断队列有没有元素：

```java
while (!queue.isEmpty()) {

}
```

这个写法以后 BFS 会疯狂出现。

意思：

> 只要队列里还有节点，就继续处理。

---

# 9. size()

```java
int size = queue.size();
```

表示：

> 当前 Queue 中有多少元素。

这在**二叉树层序遍历中特别重要**。

后面你马上会看到为什么。

---

# 10. Queue 最基础遍历

例如：

```java
Queue<Integer> queue = new ArrayDeque<>();

queue.offer(10);
queue.offer(20);
queue.offer(30);

while (!queue.isEmpty()) {

    int num = queue.poll();

    System.out.println(num);
}
```

过程：

```text
queue：

10 → 20 → 30

↓

poll()

拿出10

queue：

20 → 30

↓

poll()

拿出20

queue：

30

↓

poll()

拿出30

queue：

空

↓

while结束
```

输出：

```text
10
20
30
```

这就是 Queue 最基本的使用方式：

```java
while (!queue.isEmpty()) {

    xxx = queue.poll();

    // 处理 xxx
}
```

---

# 11. 为什么 Queue 特别适合二叉树层序遍历？

假设：

```text
        3
       / \
      9   20
         /  \
        15   7
```

所谓**层序遍历**就是：

```text
第0层：

3

第1层：

9 20

第2层：

15 7
```

访问顺序：

```text
3 → 9 → 20 → 15 → 7
```

Queue 恰好可以保证：

> **先进入队列的上一层节点，会先被处理；处理它们时，再把下一层节点放到队尾。**

---

# 12. 从 root 开始

首先：

```java
Queue<TreeNode> queue = new ArrayDeque<>();

queue.offer(root);
```

队列：

```text
[3]
```

然后：

```java
TreeNode node = queue.poll();
```

取出：

```text
3
```

---

# 13. 处理 3 的左右孩子

拿到 `3` 后：

```java
if (node.left != null) {
    queue.offer(node.left);
}

if (node.right != null) {
    queue.offer(node.right);
}
```

所以：

```text
3.left  = 9
3.right = 20
```

进入队列：

```text
队头          队尾
 ↓             ↓

9 → 20
```

---

# 14. 接下来处理 9

```java
node = queue.poll();
```

得到：

```text
9
```

因为 9 没有孩子，所以不加入东西。

队列剩：

```text
20
```

---

# 15. 接下来处理 20

```java
node = queue.poll();
```

得到：

```text
20
```

然后：

```java
queue.offer(20.left);
queue.offer(20.right);
```

于是：

```text
15 → 7
```

---

最终顺序：

```text
3
↓
9
↓
20
↓
15
↓
7
```

恰好就是：

> 层序遍历。

---

# 16. 最基础的二叉树 BFS 模板

先不考虑“每层分开”，最简单：

```java
public void bfs(TreeNode root) {

    if (root == null) {
        return;
    }

    Queue<TreeNode> queue = new ArrayDeque<>();

    queue.offer(root);

    while (!queue.isEmpty()) {

        TreeNode node = queue.poll();

        System.out.println(node.val);

        if (node.left != null) {
            queue.offer(node.left);
        }

        if (node.right != null) {
            queue.offer(node.right);
        }
    }
}
```

这个模板建议你先理解到能默写。

核心只有：

```java
queue.offer(root);

while (!queue.isEmpty()) {

    TreeNode node = queue.poll();

    if (node.left != null) {
        queue.offer(node.left);
    }

    if (node.right != null) {
        queue.offer(node.right);
    }
}
```

---

# 17. 但是 LeetCode 102 要求“分层”

LeetCode 102 返回：

```text
[
    [3],
    [9,20],
    [15,7]
]
```

而不是：

```text
[3,9,20,15,7]
```

所以问题来了：

> **怎么知道当前 Queue 中哪些节点属于这一层？**

这就是：

```java
queue.size();
```

最重要的用途。

---

# 18. `queue.size()` 为什么能表示当前层节点数？

还是：

```text
        3
       / \
      9   20
         /  \
        15   7
```

最开始：

```text
queue：

[3]
```

所以：

```java
int size = queue.size();
```

得到：

```text
size = 1
```

说明：

> 当前这一层有 1 个节点。

---

处理完 3，把：

```text
9
20
```

加入。

于是：

```text
queue：

[9,20]
```

下一轮：

```java
size = queue.size();
```

得到：

```text
2
```

所以：

> 当前层有两个节点。

---

处理：

```text
9
20
```

期间把：

```text
15
7
```

加入队尾。

最后：

```text
queue：

[15,7]
```

下一轮：

```text
size = 2
```

于是：

> 第三层两个节点。

---

# 19. 所以层序遍历会出现“双重循环”

外层：

```java
while (!queue.isEmpty()) {
```

负责：

> 一层一层处理。

内层：

```java
for (int i = 0; i < size; i++) {
```

负责：

> 把当前这一层的所有节点处理完。

结构：

```java
while (!queue.isEmpty()) {

    int size = queue.size();

    for (int i = 0; i < size; i++) {

        TreeNode node = queue.poll();

        ...
    }
}
```

这个就是层序遍历最核心的模板。

---

# 20. 一个超级重要的细节：size 必须提前保存

一定要：

```java
int size = queue.size();

for (int i = 0; i < size; i++) {
```

不要写：

```java
for (int i = 0; i < queue.size(); i++) {
```

为什么？

因为你在循环里面会：

```java
queue.poll();
```

同时还会：

```java
queue.offer(node.left);
queue.offer(node.right);
```

所以：

```java
queue.size()
```

一直在变化！

---

# 21. 举个例子

开始：

```text
queue：

[3]

size = 1
```

处理 `3`：

```java
poll()
```

然后加入：

```text
9
20
```

queue 变成：

```text
[9,20]
```

现在：

```java
queue.size()
```

已经从：

```text
1
```

变成：

```text
2
```

但是 `9、20` 属于：

> **下一层。**

我们不能这一轮顺便处理掉。

所以必须在处理当前层之前：

```java
int size = queue.size();
```

把：

> “这一层有多少节点”

固定下来。

这个知识点非常重要。

---

# 22. LeetCode 102 标准结构

现在加入你刚学的 `List`：

```java
public List<List<Integer>> levelOrder(TreeNode root) {

    List<List<Integer>> result = new ArrayList<>();

    if (root == null) {
        return result;
    }

    Queue<TreeNode> queue = new ArrayDeque<>();

    queue.offer(root);

    while (!queue.isEmpty()) {

        int size = queue.size();

        List<Integer> level = new ArrayList<>();

        for (int i = 0; i < size; i++) {

            TreeNode node = queue.poll();

            level.add(node.val);

            if (node.left != null) {
                queue.offer(node.left);
            }

            if (node.right != null) {
                queue.offer(node.right);
            }
        }

        result.add(level);
    }

    return result;
}
```

---

# 23. 这段代码和你之前学的 List 正好结合起来

外层答案：

```java
List<List<Integer>> result = new ArrayList<>();
```

例如最终：

```text
[
    [3],
    [9,20],
    [15,7]
]
```

每一层：

```java
List<Integer> level = new ArrayList<>();
```

例如：

```text
第一层：

level = [3]
```

第二层：

```text
level = [9,20]
```

第三层：

```text
level = [15,7]
```

然后：

```java
result.add(level);
```

所以：

```text
Queue<TreeNode>
      ↓
负责“哪些节点接下来需要访问”

List<Integer> level
      ↓
负责“当前层的答案”

List<List<Integer>> result
      ↓
负责“整棵树的答案”
```

这三个容器职责完全不同。

---

# 24. 手动跑一次

树：

```text
        3
       / \
      9   20
         /  \
        15   7
```

开始：

```text
result = []

queue = [3]
```

---

### 第一轮 while

```java
size = queue.size();
```

得到：

```text
size = 1
```

创建：

```text
level = []
```

poll：

```text
3
```

加入：

```text
level = [3]
```

3 的左右孩子入队：

```text
queue = [9,20]
```

当前层处理完：

```java
result.add(level);
```

于是：

```text
result：

[
  [3]
]
```

---

### 第二轮 while

现在：

```text
queue = [9,20]
```

提前：

```text
size = 2
```

创建：

```text
level = []
```

处理 9：

```text
level = [9]

queue = [20]
```

处理 20：

```text
level = [9,20]
```

把 15、7 加入：

```text
queue = [15,7]
```

保存：

```text
result：

[
  [3],
  [9,20]
]
```

---

### 第三轮

```text
queue = [15,7]

size = 2
```

处理：

```text
15
7
```

得到：

```text
level = [15,7]
```

没有孩子再加入。

queue：

```text
[]
```

最终：

```text
[
    [3],
    [9,20],
    [15,7]
]
```

---

# 25. Queue API 为什么有两套名字？

你可能以后会看到：

```java
offer()
poll()
peek()
```

也可能看到：

```java
add()
remove()
element()
```

简单知道即可：

| 推荐刷题使用 | 类似操作 |
|---|---|
| `offer(x)` | `add(x)` |
| `poll()` | `remove()` |
| `peek()` | `element()` |

主要区别是：

> 队列操作失败/为空时，一套倾向于返回特殊值，另一套倾向于抛异常。

对刷 LeetCode，我建议统一：

```java
offer()
poll()
peek()
```

这样就够了。

---

# 26. 为什么推荐 ArrayDeque？

你可能在题解里经常看到：

```java
Queue<TreeNode> queue = new LinkedList<>();
```

这是合法的。

也可以：

```java
Queue<TreeNode> queue = new ArrayDeque<>();
```

普通 BFS 中我建议优先：

```java
Queue<TreeNode> queue = new ArrayDeque<>();
```

它很适合作为普通队列使用。

不过注意：

> `ArrayDeque` 不允许加入 `null`。

所以二叉树 BFS 不要：

```java
queue.offer(node.left);
```

不管它是不是 null。

而应该：

```java
if (node.left != null) {
    queue.offer(node.left);
}
```

右边同理。

这本身也是正常 BFS 推荐写法。

---

# 27. Queue 和你之前学的数据结构对比

你现在已经接触：

```text
ArrayList
HashSet
Queue
```

可以这样区分：

| 数据结构 | 最核心的问题 |
|---|---|
| `ArrayList` | 我要保存一串数据 |
| `HashSet` | 这个东西出现过吗？ |
| `Queue` | 谁先来，谁先处理 |

比如：

```text
List
↓
保存每层结果
```

```text
HashSet
↓
visited / 判断重复
```

```text
Queue
↓
BFS / 层序遍历
```

---

# 28. DFS 和 BFS 也可以开始区分了

你之前大量做的是二叉树递归：

```java
dfs(root.left);
dfs(root.right);
```

这属于：

> DFS：Depth First Search，深度优先搜索。

比如：

```text
        1
       / \
      2   3
     / \
    4   5
```

DFS 可能：

```text
1
↓
2
↓
4
↓
回来
↓
5
↓
回来
↓
3
```

特点：

> 一条路先往深处走。

---

层序遍历：

```text
1
↓
2 3
↓
4 5
```

属于：

> BFS：Breadth First Search，广度优先搜索。

特点：

> 一层一层往外扩。

而 Queue 正好保证：

```text
上一层先进入
↓
上一层先处理
↓
处理时把下一层放到队尾
↓
下一层之后处理
```

所以：

```text
二叉树递归 → 经常 DFS

Queue       → 经常 BFS
```

---

# 29. 你现阶段 Queue 最容易犯的几个错误

第一，不要忘记先放 root：

```java
queue.offer(root);
```

否则：

```java
while (!queue.isEmpty())
```

一开始就是 false。

---

第二，`poll()` 会删除：

```java
TreeNode node = queue.poll();
```

得到队头，同时队头离开 Queue。

---

第三，孩子要重新入队：

```java
if (node.left != null) {
    queue.offer(node.left);
}

if (node.right != null) {
    queue.offer(node.right);
}
```

否则只能处理 root。

---

第四，分层时一定提前：

```java
int size = queue.size();
```

不要：

```java
for (int i = 0; i < queue.size(); i++)
```

因为 Queue 大小在循环过程中会变化。

---

第五，注意：

```java
Queue<TreeNode>
```

里面保存的是：

> `TreeNode` 节点对象。

所以：

```java
TreeNode node = queue.poll();
```

然后才能：

```java
node.val
node.left
node.right
```

---

# 30. 最值得你背下来的 Queue 语法

```java
Queue<TreeNode> queue = new ArrayDeque<>();

queue.offer(root);       // 入队

TreeNode node =
    queue.poll();        // 出队

queue.peek();            // 看队头，不删除

queue.size();            // 当前大小

queue.isEmpty();         // 是否为空
```

真正的二叉树 BFS 模板：

```java
Queue<TreeNode> queue = new ArrayDeque<>();

queue.offer(root);

while (!queue.isEmpty()) {

    TreeNode node = queue.poll();

    if (node.left != null) {
        queue.offer(node.left);
    }

    if (node.right != null) {
        queue.offer(node.right);
    }
}
```

需要**区分每一层**时：

```java
while (!queue.isEmpty()) {

    int size = queue.size();

    for (int i = 0; i < size; i++) {

        TreeNode node = queue.poll();

        if (node.left != null) {
            queue.offer(node.left);
        }

        if (node.right != null) {
            queue.offer(node.right);
        }
    }
}
```

你现在不用继续学 Queue 的底层实现。先把 **`offer → poll → size → isEmpty`** 这四个东西练熟，然后直接去做 **LeetCode 102 二叉树的层序遍历**最合适。做完 102 后，107、199、637、515 这类题基本都是在同一个层序遍历模板上改。