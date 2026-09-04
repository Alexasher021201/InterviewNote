# Java List 05：LeetCode 回溯与 DFS 中的 List

> 目标：掌握 List 在子集、组合、排列、路径问题中的核心使用方式。

## 1. 为什么回溯特别喜欢 List

回溯过程中，当前答案的长度不断变化：

``` text
[]
[1]
[1, 2]
[1, 2, 3]
[1, 2]
[1]
[]
```

数组不方便动态增加和删除，因此通常使用：

``` java
List<Integer> path = new ArrayList<>();
```

## 2. 回溯中的三个动作

### 做选择

``` java
path.add(nums[i]);
```

### 向下递归

``` java
dfs(...);
```

### 撤销选择

``` java
path.remove(path.size() - 1);
```

因此最经典的结构是：

``` java
path.add(nums[i]);

dfs(...);

path.remove(path.size() - 1);
```

记忆：

``` text
做选择
  ↓
进入下一层
  ↓
回来
  ↓
撤销选择
```

## 3. 为什么删除最后一个

假设：

``` text
path = [1, 2, 3]
```

刚才做的选择是 `3`。

递归结束后要恢复到进入这一层之前：

``` text
[1, 2]
```

因此：

``` java
path.remove(path.size() - 1);
```

## 4. 保存答案为什么要 new ArrayList

错误：

``` java
result.add(path);
```

正确：

``` java
result.add(new ArrayList<>(path));
```

原因是整个递归过程中通常反复修改同一个 `path` 对象。

我们真正需要保存的是：

> path 在"这一刻"的内容。

所以要复制。

## 5. 子集模板

``` java
class Solution {

    List<List<Integer>> result = new ArrayList<>();
    List<Integer> path = new ArrayList<>();

    public List<List<Integer>> subsets(int[] nums) {
        dfs(nums, 0);
        return result;
    }

    private void dfs(int[] nums, int start) {

        result.add(new ArrayList<>(path));

        for (int i = start; i < nums.length; i++) {

            path.add(nums[i]);

            dfs(nums, i + 1);

            path.remove(path.size() - 1);
        }
    }
}
```

核心不是死背整段，而是理解：

``` java
result.add(new ArrayList<>(path));
```

保存当前状态。

``` java
path.add(nums[i]);
```

选择。

``` java
dfs(nums, i + 1);
```

深入。

``` java
path.remove(path.size() - 1);
```

回退。

## 6. 子集过程示意

对于：

``` text
nums = [1, 2]
```

搜索过程大致：

``` text
[]
|
+-- 选择 1 -> [1]
|             |
|             +-- 选择 2 -> [1, 2]
|
+-- 选择 2 -> [2]
```

每到一个状态就保存：

``` text
[]
[1]
[1,2]
[2]
```

## 7. 固定长度组合

例如要求选择 `k` 个数字：

``` java
void dfs(int[] nums, int start, int k) {

    if (path.size() == k) {
        result.add(new ArrayList<>(path));
        return;
    }

    for (int i = start; i < nums.length; i++) {

        path.add(nums[i]);

        dfs(nums, i + 1, k);

        path.remove(path.size() - 1);
    }
}
```

这里：

``` java
path.size() == k
```

表示已经凑够一组答案。

## 8. 排列与 used\[\]

排列和组合的区别之一：

> 排列需要考虑顺序，因此不能简单只往后选。

常见模板：

``` java
class Solution {

    List<List<Integer>> result = new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    boolean[] used;

    public List<List<Integer>> permute(int[] nums) {

        used = new boolean[nums.length];

        dfs(nums);

        return result;
    }

    private void dfs(int[] nums) {

        if (path.size() == nums.length) {
            result.add(new ArrayList<>(path));
            return;
        }

        for (int i = 0; i < nums.length; i++) {

            if (used[i]) {
                continue;
            }

            used[i] = true;
            path.add(nums[i]);

            dfs(nums);

            path.remove(path.size() - 1);
            used[i] = false;
        }
    }
}
```

这里 `used[i]` 表示：

> 当前这条路径中，nums\[i\] 是否已经使用。

## 9. 为什么不总用 contains

初学时可能写：

``` java
if (!path.contains(nums[i])) {
    // ...
}
```

但 `contains()` 通常是 `O(n)`。

如果题目需要根据**下标**判断某元素是否使用，常用：

``` java
boolean[] used;
```

查询是 `O(1)`，语义也更加准确。

## 10. 树路径中也常用 List

例如 DFS 保存从根节点到当前节点的路径：

``` java
void dfs(TreeNode root) {

    if (root == null) {
        return;
    }

    path.add(root.val);

    dfs(root.left);
    dfs(root.right);

    path.remove(path.size() - 1);
}
```

依然是：

``` text
进入节点 -> add
递归子树
离开节点 -> remove
```

## 11. 回溯 List 四句口诀

对刷题非常值得形成肌肉记忆：

``` java
path.add(x);                         // 做选择

dfs(...);                            // 递归

path.remove(path.size() - 1);        // 撤销选择

result.add(new ArrayList<>(path));   // 保存当前答案
```

真正需要理解的是：

> `path` 是当前搜索路径，`result` 是已经确定要保留下来的历史结果。
