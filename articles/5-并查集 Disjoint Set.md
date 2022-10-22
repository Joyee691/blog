# 并查集 Disjoint Set

> 本文共 1419 字，建议阅读时间 15 min



## 前言

[并查集（Disjoint-set data structure）](https://en.wikipedia.org/wiki/Disjoint-set_data_structure)是一种数据结构，常常被用来表示一系列不重复元素的集合。

它支持三个操作：

- 查询（Find）：查询某个元素属于哪一个集合，返回一个代表该集合的元素。常用来判断两个元素是否属于同一个集合
- 合并（Union）：将两个集合合并为一个集合
- 添加（Add）：添加一个新集合，其中只有一个新元素。由于我们默认一开始的一系列不重复元素各自都有一个集合，所以这个操作可以忽略

本文将从一个实际的例子入手，逐步实现并查集的算法，并在最后进一步给出它的优化版本。

## 一个例子

假设某个地区有 6 个部落，部落之间各自为政，看似没什么交集。

![5-1](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/5-1.svg)

但是有一天，有个情报机关得到了一些线索：`{[1,3], [1,2], [0,5], [5,4], [2,0]}`  其中数组表示两个部落有联盟。

我们需要建立一套算法来判断两个部落之间的联盟关系。

<br />

首先我们需要一个数组，数组长度为部落的数量，初始值都为自己的下标，表示每个部落的老大是自己。

![5-2](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/5-2.svg)

我们的初始化代码可以写成：

```cpp
class UnionFind {
private:
  vector<int> root;
public:
  UnionFind(int size): root(size) {
    for(int i = 0;i < size;i++) {
      root[i] = i;
    }
  }
};
```

 <br />

接着，我们需要给结盟的部落之间指定一个首领，我们先假设数组的第一个的部落为联盟的首领。

以 `[1,3]` 为例，我们可以用 `root[3] = 1` 表示部落三听令于部落一，并给部落三画一条指向向部落一的箭头：

![5-3](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/5-3.svg)

<br />

顺着上面的思路，我们把剩下的线索都安排上，可以得到如下图示：

![5-4](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/5-4.svg)

至此，我们可以得到第一个版本的并查集代码：

```cpp
class UnionFind {
private:
  vector<int> root;
public:
  UnionFind(int size): root(size) {
    for(int i = 0;i < size;i++) {
      root[i] = i;
    }
  }
  
  // 找老大，只有老大是指向自己的
  int find(int x) {
    if (root[x] == x) {
      return x;
    }
    return find(root[x]);
  }

  // 简单粗暴
  void merge(int x, int y) { root[y] = x; }
};
```

<br />

但是上面的代码有一点问题，如果把最后的图看成一颗树的话，可以看到非常明显的子树高度不平衡，这种不平衡在数据量大的时候会更加明显，且会导致搜索效率的下降

## 小改进

我们可以通过一个思路上的小改进来缓解左右子树不平衡的问题：**只让老大跟随老大**

还是上面的例子，只是在进行到`[5, 4]` 这个步骤的时候，我们要做两件事：

1. 先找看看部落五的联盟有没有老大，在这个例子，老大是部落零
2. 找一下部落四的老大，在这个例子，部落四的老大就是自己
3. 将部落四指向部落零

如此一来图会变成这样：

![5-5](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/5-5.svg)

<br />

顺着上面的思路，最后的图会变成这样：

![5-6](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/5-6.svg)

改进版的并查集代码如下：

```cpp
class UnionFind {
private:
  vector<int> root;
public:
  UnionFind(int size): root(size) {
    for(int i = 0;i < size;i++) {
      root[i] = i;
    }
  }
  
  int find(int x) {
    if (root[x] == x) {
      return x;
    }
    return find(root[x]);
  }

  // 将部落 y 的联盟老大指向部落 x 的联盟老大
  void merge(int x, int y) { root[find(y)] = find(x); }
};
```

完美了吗？其实还没有😎



## 最终优化

我们是时候来考虑另一个例子了：

![5-7](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/5-7.svg)

在这个例子中，我们想要把两个集合合并，会有两种方法：

- 把 0 指向 1
- 把 1 指向 0

那么哪一个方法会比较好呢？

要回答这个问题，必须先知道并查集的性能瓶颈在哪里，可以看到，并查集最常用的两个操作分别是 `merge` 和 `find`

其中，`merge` 操作内部多次调用了 `find`，说明 `find` 操作最有可能造成性能瓶颈

由于 `find` 操作本质上是一个向上溯源的方法，所以我们可以认为，**并查集的最长子树长度决定了这个算法的效率**

综上所述，我们会选择把 0 指向 1 的这个方法，因为这样一来最长子树长度为 4；而另一种方法为 5

<br /> 为了实现上面的思路，我们需要一个另外的数组 `rank`，用来记录当前的深度，具体实现如下：

```cpp
class UnionFind {
 private:
  vector<int> root;
  // 一个数组用来记录子树的长度
  vector<int> rank;

 public:
  // 子树长度初始化为 1
  UnionFind(int size) : root(size), rank(size) {
    for (int i = 0; i < size; i++) {
      root[i] = i;
      rank[i] = 1;
    }
  }

  int find(int x) {
    if (root[x] == x) {
      return x;
    }
    return find(root[x]);
  }

  void merge(int x, int y) {
    int rootX = find(x);
    int rootY = find(y);
    if (rootX != rootY) {
      // 如果 x 的子树长度比 y 的长，就把 y 的箭头指向 x
      // 如果子树长度相等就随便了
      if (rank[rootX] >= rank[rootY]) {
        root[rootY] = rootX;
      } else {
        root[rootX] = root[rootY];
      }
      // 如果两个子树高度相同，合并之后当作根的那个子树高度必 +1，要更新一下
      if (rank[rootX] == rank[rootY]) {
        rank[rootX]++;
      }
    }
  }
};

```

## Reference

https://zhuanlan.zhihu.com/p/93647900

https://en.wikipedia.org/wiki/Disjoint-set_data_structure

