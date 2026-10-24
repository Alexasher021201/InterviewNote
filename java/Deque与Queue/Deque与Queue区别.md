# Java Deque 与 Queue 区别｜复习笔记

## 1. 核心关系

- `Queue<E>`：队列接口，提供入队、出队、查看队头等操作。
- `Deque<E>`：双端队列接口，继承 `Queue<E>`，可以操作头部和尾部，也提供栈操作。
- `ArrayDeque<E>`：实现 `Deque<E>` 的类，因此也能作为 `Queue<E>` 使用。

普通队列通常先进先出（FIFO），栈后进先出（LIFO）。注意：`Queue` 并不保证所有实现都先进先出，例如 `PriorityQueue` 按优先级出队。

## 2. 左边的变量类型与右边的 new

```java
Queue<Integer> queue = new ArrayDeque<>();
Deque<Integer> deque = new ArrayDeque<>();
```

| 部分 | 含义 |
| --- | --- |
| 左边的 `Queue<Integer>` / `Deque<Integer>` | 变量的声明类型，决定编译器允许调用哪些方法 |
| 中间的 `queue` / `deque` | 变量名，可以自由命名，不决定是栈还是队列 |
| 右边的 `new ArrayDeque<>()` | 创建实际的 `ArrayDeque` 对象，具体方法行为由对象的实现决定 |

**两行代码创建的是同一种对象；左边的类型不同，可直接调用的方法范围不同。**

```java
queue.offer(1);       // 可以：Queue 中有此方法
queue.poll();         // 可以
// queue.push(1);     // 编译错误：Queue 中没有 push
// queue.offerFirst(1); // 编译错误：Queue 中没有 offerFirst

deque.offer(1);       // 可以：继承自 Queue
deque.offerFirst(2);  // 可以：Deque 提供的双端操作
deque.push(3);        // 可以：Deque 提供的栈操作
```

子接口类型可以赋值给父接口类型，下面两个变量引用同一个对象：

```java
Deque<Integer> deque = new ArrayDeque<>();
Queue<Integer> queue = deque; // 可以；通过 queue 仍不能调用 push、pop
```

## 3. Queue 的常用方法

| 操作 | 失败时抛异常 | 失败时返回特殊值 |
| --- | --- | --- |
| 添加元素 | `add(e)` | `offer(e)`：无法因容量限制添加时返回 `false` |
| 取出并删除队头 | `remove()` | `poll()`：空时返回 `null` |
| 查看队头，不删除 | `element()` | `peek()`：空时返回 `null` |

“返回特殊值”不代表任何情况都不抛异常，例如非法参数仍可能导致异常。

常用辅助方法：`size()` 获取元素数量，`isEmpty()` 判断是否为空。

## 4. Deque 增加的双端方法

`First` 表示头部，`Last` 表示尾部；以下方法的空队列、容量限制处理规则与上表对应。

| 操作 | 头部 | 尾部 |
| --- | --- | --- |
| 添加，失败抛异常 | `addFirst(e)` | `addLast(e)` |
| 添加，失败返回 `false` | `offerFirst(e)` | `offerLast(e)` |
| 取出并删除，空时抛异常 | `removeFirst()` | `removeLast()` |
| 取出并删除，空时返回 `null` | `pollFirst()` | `pollLast()` |
| 查看，空时抛异常 | `getFirst()` | `getLast()` |
| 查看，空时返回 `null` | `peekFirst()` | `peekLast()` |

常用方法的对应关系：

```java
offer(e)  // 等同于 offerLast(e)：尾部入队
poll()    // 等同于 pollFirst()：头部出队
peek()    // 等同于 peekFirst()：查看头部
push(e)   // 等同于 addFirst(e)：头部入栈
pop()     // 等同于 removeFirst()：头部出栈，空时抛异常
```

## 5. 当作队列还是栈，取决于如何操作

```java
// 队列：尾部入、头部出，先进先出
Queue<Integer> queue = new ArrayDeque<>();
queue.offer(1);
queue.offer(2);
System.out.println(queue.poll()); // 1
System.out.println(queue.poll()); // 2

// 栈：头部入、头部出，后进先出
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);
stack.push(2);
System.out.println(stack.pop());  // 2
System.out.println(stack.pop());  // 1
```

**上面的队列也可以声明为 `Deque<Integer> queue = new ArrayDeque<>();`，继续使用 `offer()`、`poll()`，行为相同。**

## 6. 复习重点：Queue 类型的变量能直接当栈吗？

**在这两种声明类型之间：`Deque` 可以直接使用栈操作，`Queue` 不可以。** 即使右边都是 `new ArrayDeque<>()`，`Queue` 类型的变量也不能直接调用 `push()`、`pop()`。

这不意味着 Java 中只有 `Deque` 类型能表示栈；其他类型也能实现栈。这里强调的是这两个接口暴露的方法不同。

| 使用需求 | 常见声明 | 常用操作 |
| --- | --- | --- |
| 普通队列，如 BFS | `Queue<Integer> queue = new ArrayDeque<>();` | `offer()`、`poll()`、`peek()` |
| 栈，如括号匹配 | `Deque<Integer> stack = new ArrayDeque<>();` | `push()`、`pop()`、`peek()` |
| 双端操作 | `Deque<Integer> deque = new ArrayDeque<>();` | 带 `First` / `Last` 的方法 |

易错点：`ArrayDeque` 不允许存入 `null`；`peek()` 不删除元素，`poll()`、`pop()` 会删除元素；不要混用入队和入栈方法，否则容易弄错顺序。
