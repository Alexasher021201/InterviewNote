**LeetCode 39、40：组合总和与回溯笔记**

**一、先理解两道题的区别**

两道题都要求：从 `candidates` 中选出一些数字，使它们的和等于 `target`，返回所有不重复的组合。

| 对比 | 39：组合总和 | 40：组合总和 II |
|---|---|---|
| 输入中是否存在重复数字 | 不存在 | 可能存在 |
| 每个数组位置能使用几次 | 可以重复使用 | 最多一次 |
| 选择当前数字后，递归的下标 | `index` | `index + 1` |
| 去重方式 | 保持下标不往回走即可 | 排序后，在搜索过程中跳过重复分支 |

例如：

```text
39：candidates = [2,3]，target = 7
可以得到 [2,2,3]，同一个 2 可以使用多次。

40：candidates = [2,3]，target = 7
没有答案，数组中只有一个 2。

40：candidates = [2,2,3]，target = 7
可以得到 [2,2,3]，两个 2 来自不同位置。
```

组合不考虑顺序，`[2,2,3]` 和 `[3,2,2]` 算同一个答案。

---

**二、回溯函数里的变量是什么意思**

按照我原来的“选或跳过”写法：

```java
dfs(candidates, target, result, temp, index)
```

- `target`：还需要凑出的数值。
- `temp`：当前已经选择的数字。
- `index`：当前正在考虑的数组位置。
- `result`：保存所有找到的答案。

每次递归都在决定：

```text
选择 candidates[index]
或者
跳过 candidates[index]
```

只向当前或后面的下标搜索，可以避免把同一个组合按不同顺序重复生成。

回溯的基本结构：

```java
temp.add(x);                    // 做选择
dfs(...);                       // 继续搜索
temp.remove(temp.size() - 1);    // 撤销选择
```

**撤销选择，是为了让后续分支从原来的状态出发。**

---

**三、LeetCode 39 中我犯的错误**

**错误 1：直接保存 `temp`**

原来的代码：

```java
result.add(temp);
```

`temp` 是一个不断被修改的列表。这行代码保存的是它的引用，后续增删 `temp`，已经保存的答案也会跟着变化。

正确写法：

```java
result.add(new ArrayList<>(temp));
```

可以理解为：**找到答案时，给当前路径拍一张快照。**

**错误 2：添加数字后，没有撤销选择**

原来的代码：

```java
if (candidates[index] == target) {
    temp.add(candidates[index]);
    result.add(temp);
    target = target - candidates[index];
}
```

这里添加的数字没有被删除，后续分支会带着这个数字继续搜索，导致路径错误。

同时，修改当前层的 `target`，会影响当前层后面执行的分支。

更容易维护的写法是：

```java
temp.add(candidates[index]);
dfs(candidates, target - candidates[index], ...);
temp.remove(temp.size() - 1);
```

这里 `target - candidates[index]` 作为参数传入下一层，**当前层的 `target` 保持原值**。Java 的 `int` 按值传递，因此不需要恢复它；共享的 `temp` 则需要手动恢复。

**错误 3：结束条件没有统一处理答案**

原来一开始写了：

```java
if (index >= candidates.length || target <= 0) {
    return;
}
```

但 `target == 0` 恰好代表找到了答案，应该先保存，再返回：

```java
if (target == 0) {
    result.add(new ArrayList<>(temp));
    return;
}
```

这样就不需要专门判断 `candidates[index] == target` 了。

39 的完整写法：

```java
class Solution {
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        List<List<Integer>> result = new ArrayList<>();
        dfs(candidates, target, result, new ArrayList<>(), 0);
        return result;
    }

    private void dfs(
        int[] candidates,
        int target,
        List<List<Integer>> result,
        List<Integer> temp,
        int index
    ) {
        if (target == 0) {
            result.add(new ArrayList<>(temp));
            return;
        }

        if (index >= candidates.length) {
            return;
        }

        // 选择当前数字
        if (candidates[index] <= target) {
            temp.add(candidates[index]);

            // 可以重复使用，所以仍然传 index
            dfs(candidates, target - candidates[index],
                result, temp, index);

            temp.remove(temp.size() - 1);
        }

        // 跳过当前数字
        dfs(candidates, target, result, temp, index + 1);
    }
}
```

---

**四、LeetCode 40 中我犯的错误**

我已经正确地做了两件事：

```java
result.add(new ArrayList<>(temp)); // 保存副本
dfs(..., index + 1);              // 每个位置只能使用一次
```

但我用 `HashSet` 在找到答案后去重：

```java
if (!set.contains(temp)) {
    result.add(new ArrayList<>(temp));
    set.add(new ArrayList<>(temp));
}
```

**问题是：结果去重，并没有减少重复的搜索。**

例如输入 30 个 `1`，`target = 30`：

```text
每个位置都有“选择”和“跳过”两种情况。
原来的搜索会枚举约 2^30，也就是 10.7 亿种选择方案。
```

虽然答案只有一个，但大量凑不够 30 的分支仍然会被搜索，因此超时。

解决方法：**排序后，在搜索过程中去重。**

保留“选或跳过”的结构时：

- 选择当前数字：下一层考虑 `index + 1`，允许继续选择后面相同的数字。
- 跳过当前数字：一次跳过后面所有相同的数字。

例如：

```text
[1a, 1b, 1c, 2]
```

`a、b、c` 只是为了区分三个位置，它们的值都是 `1`。

```text
选择 1a → 后面还可以选择 1b、1c，组成多个 1。

跳过 1a → 直接去考虑 2。
```

因为只要组合需要一个 `1`，选择最前面的那个就能代表这种组合；从后面的 `1` 重新开始，会产生重复结果。

40 的完整写法：

```java
class Solution {
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        Arrays.sort(candidates);

        List<List<Integer>> result = new ArrayList<>();
        dfs(candidates, target, result, new ArrayList<>(), 0);
        return result;
    }

    private void dfs(
        int[] candidates,
        int target,
        List<List<Integer>> result,
        List<Integer> temp,
        int index
    ) {
        if (target == 0) {
            result.add(new ArrayList<>(temp));
            return;
        }

        if (index >= candidates.length) {
            return;
        }

        // 已经排序，当前数字太大，后面的也都太大
        if (candidates[index] > target) {
            return;
        }

        // 选择当前数字，每个位置只能使用一次
        temp.add(candidates[index]);

        dfs(candidates, target - candidates[index],
            result, temp, index + 1);

        temp.remove(temp.size() - 1);

        // 跳过当前数字，以及后面所有相同数字
        int nextIndex = index + 1;
        while (nextIndex < candidates.length
                && candidates[nextIndex] == candidates[index]) {
            nextIndex++;
        }

        dfs(candidates, target, result, temp, nextIndex);
    }
}
```

如果使用 `for` 循环枚举下一次选哪个数字，对应的去重条件是：

```java
if (i > start && candidates[i] == candidates[i - 1]) {
    continue;
}
```

其中 `i > start` 很关键：**同一层跳过重复数字，下一层仍然可以选择另一个相同数字。** 所以 `[1,1,6]` 这样的组合能够正常生成。

---

**五、以后写回溯时检查这几件事**

1. **什么时候找到答案？**  
   本题是 `target == 0`，此时保存副本并返回。

2. **路径有没有恢复？**  
   每次 `temp.add(...)` 后，递归结束都要对应一次 `temp.remove(...)`。

3. **元素能不能重复使用？**  
   能重复使用传 `index`；每个位置只能用一次传 `index + 1`。

4. **重复输入会不会产生重复分支？**  
   40 要排序，并在搜索过程中跳过重复分支。

5. **剪枝有没有依据？**  
   排序后，如果当前数字已经大于剩余的 `target`，后面更大的数字也不用搜索了。