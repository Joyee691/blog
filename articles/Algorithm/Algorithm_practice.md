# 算法练习——科学的准备算法题

# 递归思想

## 回溯 Backtracking

> 回溯是一种普遍用于 DFS 与 BFS 的技巧
>
> 其本质是：走不通就回头

**回溯过程**：

1. 构造空间树；
2. 进行遍历；
3. 如遇到边界条件，即不再向下搜索，转而搜索另一条链；
4. 达到目标条件，输出结果

**算法模版**：

```js
function backtrack(path, params) {
    if (baseCase) {
        // store or print result
        return;
    }

    for (let i = start; i < choices.length; i++) {
        // choose
        path.push(choices[i]);

        // explore
        backtrack(path, updatedParams);

        // un-choose
        path.pop();
    }
}
```

**复杂度分析**：

https://zhuanlan.zhihu.com/p/448969860

# 二分搜索 Binary search

> 主要目的是用来在一个有序数组中查找某一元素

**算法模版**：

```js
var binarySearch = function(nums, target) {
    let left = 0
    let right = nums.length

    while(left < right) {
        let mid = Math.floor((left + right) / 2)

        if(nums[mid] < target) {
            left = mid + 1
        } else {
            right = mid
        }
    }

    return left
};
```

**复杂度分析**：

- 时间复杂度：`O(logn)`
- 空间复杂度：`O(1)`

# 动态规划 Dynamic Programming

> 动态规划是一种通过把原问题分解为相对简单的子问题的方式求解复杂问题的方法
>
> 动态规划并不是某种具体的算法，而是一种解决特定问题的方法

动态规划的核心思想就是**拆分子问题，记住过往，减少重复计算**

## 动态规划与递归的区别

- 递归算法一般是**自顶向下**的：从一个规模较大的原问题向下逐渐分解规模，最后逐层返回答案
- 动态规划一般是**自底向上**：直接从最底下，最简单，问题规模最小的状态开始，逐步推到我们想要解决的问题

## 什么问题适合用动态规划？

1. 该问题可以拆分成多个子问题；且该问题的最优解包含的子问题的解也是最优的，即 **最优子结构性质**
2. 该问题拆分出来的多个子问题中，包含有重复的子问题，即 **子问题重叠性质**
3. 某阶段状态一旦确定，就不受这个状态以后决策的影响，即 **无后效性**

当问题同时满足以上三点要求，即可使用动态规划来求解

## 动态规划的解题步骤

1. 把问题划分为多个阶段（每个阶段包含多个子问题）
2. 确定每个阶段的状态，确定**初始状态**
3. 寻找每个阶段之间的**状态转移方程**，确定状态转移的终止条件
4. 根据初始状态与状态转移方程，持续计算知道满足终止条件

## 例子——爬楼梯

[Leetcode 70](https://leetcode.com/problems/climbing-stairs)

第一步，划分问题：

- 我们用 `f(n)` 当成登上台阶 n 的状态
- 对于任意的台阶 n 来说，能跳上它之前，人只有两种可能：在台阶 `n-1` 上（记作 `f(n-1)` ）；在台阶 `n-2` 上（记作 `f(n-2)` ）
- 至此，我们把 `f(n)` 划分成为了 `f(n-1)` 与 `f(n-2)` 

第二步，确定初始状态：

- 在这题的例子中，初始状态有两个：`f(1)=1`；`f(2)=2`
- 解释：登上一个台阶，只能有一种方法、登上两个台阶可以：一次上一个，上两次；一次上两个（两种方法）

第三步，状态转移方程：

- `f(n) = f(n-1) + f(n-2)`
- 解释：登上第 `n-1` 个台阶的可能方法数 + 登上第 `n-2` 个台阶可能的方法数 = 登上第 `n` 个台阶的可能方法数

第四步，计算：

```js
var climbStairs = function (n) {
    const dp = [1, 2]
    for (let i = 2; i < n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2]
    }
    return dp[n - 1]
};
```

# 贪心思想

贪心的核心就是**每一步行动总是按某种指标选取最优的操作**

可想而之，它的缺点就是并非所有情况都能取得最优解

它的适用场景在**有最优子结构的问题**

例子：[121. Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)

# 数据结构

## 二叉树

**数据结构**：

```js
function TreeNode(val, left, right) {
    this.val = (val===undefined ? 0 : val)
    this.left = (left===undefined ? null : left)
    this.right = (right===undefined ? null : right)
}
```

### 树的遍历

树的遍历一般有四种：

- 前序遍历
- 中序遍历
- 后序遍历
- 层序遍历

#### 前序遍历

> 遍历顺序：根节点 -> 左子树 -> 右子树

**算法模版**：

```js
var preOrderTraversal = function(root) {
    const res = []
    const dfs = (node) => {
        if(!node) {
            return
        }
        res.push(node.val)
        dfs(node.left)
        dfs(node.right)
    }
    dfs(root)
    return res
};
```

#### 中序遍历

> 遍历顺序：左子树 -> 根节点 -> 右子树

**算法模版**：

```js
var inOrderTraversal = function(root) {
    const res = []
    const dfs = (node) => {
        if(!node) {
            return
        }
        dfs(node.left)
      	res.push(node.val)
        dfs(node.right)
    }
    dfs(root)
    return res
};
```

#### 后序遍历

> 遍历顺序：左子树 -> 右子树 -> 根节点

**算法模版**：

```js
var postOrderTraversal = function(root) {
    const res = []
    const dfs = (node) => {
        if(!node) {
            return
        }
        dfs(node.left)
        dfs(node.right)
      	res.push(node.val)
    }
    dfs(root)
    return res
};
```

#### 层序遍历

> 遍历顺序：从根节点从上往下逐层遍历，在同一层，按从左到右的顺序对结点逐个访问

**算法模版**：

```js
var layeredTraversal = function (root) {
    const queue = [root]
    const res = []
    while (queue.length > 0) {
        const node = queue.shift()
        if (!node) {
            continue
        }
        res.push(node.val)
        queue.push(node.left, node.right)
    }
    return res
};
```

## 堆

堆的本质是一颗树

与树的区别在于，堆要求任意节点的值皆大于/小于/等于其父节点的值

**每个节点的键值都大于等于其父亲键值**的堆叫做小顶堆；否则称之为大顶堆

常见的堆，我们称之为二叉堆，其对应的是二叉树

### 堆的实现

```js
class MyHeap {
  	// compare 方法决定了这个堆的性质（小顶堆/大顶堆）
    constructor(compare) {
        this.heap = []
        this.compare = compare
    }

    peek() {
        return this.heap[0]
    }

    isEmpty() {
        return this.heap.length === 0
    }

    push(item) {
        this.heap.push(item)
        this.heapifyUp(this.heap.length - 1)
    }

    pop() {
        if (this.heap.length <= 1) {
            return this.heap.pop()
        }

        const res = this.heap[0]
        this.heap[0] = this.heap.pop()
        this.heapifyDown(0)
        return res
    }

    heapifyUp(index) {
        let parent = (index - 1) >> 1

        if (parent >= 0 && this.compare(this.heap[index], this.heap[parent]) < 0) {
            // let temp = this.heap[index]
            // this.heap[index] = this.heap[parent]
            // this.heap[parent] = temp
            [this.heap[index], this.heap[parent]] = [this.heap[parent], this.heap[index]]
            this.heapifyUp(parent)
        }
    }

    heapifyDown(index) {
        const left = index * 2 + 1
        const right = index * 2 + 2
        let competitive = index

        if (left < this.heap.length && this.compare(this.heap[left], this.heap[competitive]) < 0) {
            competitive = left
        }
        if (right < this.heap.length && this.compare(this.heap[right], this.heap[competitive]) < 0) {
            competitive = right
        }

        if (competitive !== index) {
            [this.heap[competitive], this.heap[index]] = [this.heap[index], this.heap[competitive]]
            this.heapifyDown(competitive)
        }
    }
}
```

