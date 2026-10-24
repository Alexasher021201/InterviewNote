# Java 使用 Deque 建立栈

## 1. 创建栈

```java
import java.util.ArrayDeque;
import java.util.Deque;

Deque<Integer> stack = new ArrayDeque<>();
```

- `Deque<Integer>`：变量的声明类型，提供 `push()`、`pop()`、`peek()` 等方法。
- `stack`：变量名，表示准备把它当栈使用；名字本身不决定行为。
- `new ArrayDeque<>()`：创建实际保存元素的对象。
- `Integer`：元素类型；泛型不能写基本类型 `int`，存取时会自动装箱、拆箱。

栈的规则是**后进先出（LIFO）**：最后压入的元素最先弹出。使用 `Deque` 的栈方法时，**头部就是栈顶**。

## 2. 常用方法速查

| 方法 | 含义 | 返回值 | 栈为空时 |
| --- | --- | --- | --- |
| `stack.push(e)` | 压入栈顶 | `void`，无返回值 | 正常添加 |
| `stack.pop()` | 弹出并删除栈顶 | 被删除的元素 | 抛出 `NoSuchElementException` |
| `stack.peek()` | 查看栈顶，不删除 | 栈顶元素 | 返回 `null` |
| `stack.isEmpty()` | 判断是否为空 | `boolean` | 返回 `true` |
| `stack.size()` | 获取元素数量 | `int` | 返回 `0` |

如果希望空栈取出时返回 `null`，可以使用 `stack.pollFirst()`；日常栈代码通常使用 `push()`、`pop()`、`peek()`。

## 3. 入栈、查看、出栈示例

以下示例独立阅读，均需导入 `Deque` 和 `ArrayDeque`。

```java
Deque<Integer> stack = new ArrayDeque<>();

stack.push(10);
stack.push(20);
stack.push(30);
// 从栈顶到栈底：[30, 20, 10]

System.out.println(stack.peek());    // 30，只查看，不删除
System.out.println(stack.size());    // 3

System.out.println(stack.pop());     // 30
System.out.println(stack.pop());     // 20
System.out.println(stack.pop());     // 10
System.out.println(stack.isEmpty()); // true
System.out.println(stack.peek());    // null
// stack.pop();                     // 空栈，抛异常
```

对应的双端操作：

```java
stack.push(x); // 等同于 stack.addFirst(x)
stack.pop();   // 等同于 stack.removeFirst()
stack.peek();  // 等同于 stack.peekFirst()
```

## 4. 常用处理模板

### 依次弹出所有元素

```java
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);
stack.push(2);
stack.push(3);

while (!stack.isEmpty()) {
    int value = stack.pop();
    System.out.println(value); // 依次输出 3、2、1
}
// 循环结束后，栈为空
```

### 只遍历，不弹出

```java
for (Integer value : stack) {
    System.out.println(value);
}
```

对于一直用 `push()` 入栈的 `ArrayDeque`，普通遍历顺序是**栈顶到栈底**，遍历本身不会删除元素。

## 5. 其他元素类型

```java
Deque<Character> chars = new ArrayDeque<>(); // 字符栈，如括号匹配
Deque<String> words = new ArrayDeque<>();    // 字符串栈
Deque<Integer> indices = new ArrayDeque<>(); // 保存下标，如单调栈
// Deque<TreeNode> nodes = new ArrayDeque<>(); // 定义 TreeNode 后可用于迭代 DFS
```

保存下标时，`peek()`、`pop()` 得到的是下标，要通过 `nums[index]` 访问对应数值。

## 6. 易错点

1. **用栈方法时声明为 `Deque`。** `Queue<Integer> stack = new ArrayDeque<>();` 虽然能创建对象，但不能通过这个变量直接调用 `push()`、`pop()`，因为 `Queue` 接口没有这些方法。
2. **`peek()` 不删除，`pop()` 会删除。** 弹出前通常先检查 `!stack.isEmpty()`。
3. **空栈 `peek()` 返回 `null`。** `int x = stack.peek();` 在空栈时会因自动拆箱抛出 `NullPointerException`。
4. **`ArrayDeque` 不允许存入 `null`。** `stack.push(null)` 会抛出 `NullPointerException`。
5. **栈操作保持一致。** `push()` 从头部添加；`offer()` 默认从尾部添加。写普通栈时统一使用 `push()`、`pop()`、`peek()`，避免混淆顺序。

## 7. 最小记忆模板

```java
Deque<Integer> stack = new ArrayDeque<>();
stack.push(x);              // 入栈

if (!stack.isEmpty()) {
    int top = stack.peek(); // 看栈顶
    int value = stack.pop();// 弹出栈顶
}
```

`ArrayDeque` 的 `push()`、`pop()` 为均摊 O(1)，`peek()`、`size()`、`isEmpty()` 为 O(1)；保存 n 个元素需要 O(n) 空间。
