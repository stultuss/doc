搜索树
=========================

> 学习二叉树之前需要了解链表，链表的插入和删除是非常快的，只需要改变引用即可，但是查找却很慢。不管需要查找什么数据，都需要从第一个数据往下遍历，直到找到所需要的数据，平均时间复杂度为 O(n/2)，而二叉树的数据结构就同时具有查找快和插入删除快的优点。

**大多数数据库系统及文件系统都是采用 B-Tree 或其变种 B+Tree 作为索引结构。例如 MySQL**

## 简介

这里主要介绍的是二叉搜索树，也就是**每个节点都不比它左子树的任意元素小，而且不比它的右子树的任意元素大。**

```javascript
// 节点构造函数
function Node(data, left, right) {
  this.data = data; // 节点 id
  this.left = left; // 子节点（左）
  this.right = right; // 子节点（右）
}

// 二叉树
function BST() {
  this.root = null;
}

// 塞入二叉树
BST.prototype.insert = function (data) {
  const node = new Node(data, null, null);
  if (this.root == null) {
    this.root = node;
  } else {
    let current = this.root;
    let parent;
    while (true) {
      parent = current;
      if (data < current.data) {
        current = current.left;
        if (current == null) {
          parent.left = node;
          break;
        }
      } else {
        current = current.right;
        if (current == null) {
          parent.right = node;
          break;
        }
      }
    }
  }
};

module.exports = BST;
```

## 遍历

> 这是必考知识点。

![img](https://i.loli.net/2019/07/29/5d3e58674375c14067.jpg)

概念：指从根节点出发，按照某种次序依次访问二叉树中所有节点，使得每个节点都被访问一次，且只被访问一次。二叉树的访问次序一共分为四种：

1. 前序遍历，从根节点出发，先遍历左侧子节点，遍历完再遍历右侧节点，每当遍历到一个节点，就输出节点的值。答案是：A，B，D，C，E，F
2. 中序遍历，从根节点出发，先遍历左侧子树，直到遍历完这个节点的子树，再输出这个节点，然后再遍历右侧子树。答案是：D，B，A，E，C，F
3. 后序遍历，从根节点出发，先遍历左侧子树，直到该节点左右子树都遍历完后，再输出这个节点。答案是：D，B，E，F，C，A
4. 层序遍历，从根节点出发，将根节点入队，然后遍历规则为：队列非空则出队，并将出队的左右子节点依次入队。答案是：A，B，C，D，E，F

### 深度优先搜索（DFS）

前序，中序，后序遍历都属于 DFS，它沿着树的深度遍历树的节点，尽可能深的搜索树的分支，当节点 v 所有的边都被探寻过，搜索将回溯到节点 v 的那条边的起始节点，递归这个过程，直到所有节点被访问。

```javascript
// 前序遍历(递归)
BST.prototype.preOrder = function (node, res) {
  if (!res) {
    res = [];
  }
  if (node) {
    res.push(node.data);
    this.preOrder(node.left, res);
    this.preOrder(node.right, res);
  }
  return res;
};

// 中序遍历
BST.prototype.inOrder = function (node, res) {
  if (!res) {
    res = [];
  }
  if (node) {
    this.inOrder(node.left, res);
    res.push(node.data);
    this.inOrder(node.right, res);
  }
  return res;
};

// 后序遍历
BST.prototype.postOrder = function (node, res) {
  if (!res) {
    res = [];
  }
  if (node) {
    this.postOrder(node.left, res);
    this.postOrder(node.right, res);
    res.push(node.data);
  }
  return res;
};
```

### 广度优先搜索（BFS）

层序遍历属于 BFS，一般用于求最短路径，需要依赖队列来实现。这种搜索方法不考虑结果可能的位置，而是彻底的搜索整个图，直到找到结果为止。

![自然界中的宽搜](https://gss3.bdstatic.com/7Po3dSag_xI4khGkpoWK1HF6hhy/baike/s%3D220/sign=a935710e0c33874498c5287e610fd937/adaf2edda3cc7cd9d2011b873901213fb80e91bb.jpg)

## 特殊二叉树

### 满二叉树

> 在一棵二叉树中,如果所有的分支节点都存在左子树和右子树,并且所有叶子都在同一层面上,这样的二叉树成为满二叉树.

特点：

- 满二叉树的叶子只能出现在最下一层，出现在其它层就不可能达成平衡。
- 非叶子节点的度一定是 2。
-  在同样深度的热茶树中，满二叉树的节点个数最多，叶子数最多。

### 完全二叉树

>  对于一棵具有 n 个节点的二叉树按层序编号,如果编号为 i ( 1<= i <= n) 的节点与同样深度的满二叉树中编号为 i 的节点在二叉树中位置完全相同,则这棵二叉树成为完全二叉树.

特点：

- 完全二叉树的叶子节点只能出现在最下面的两层.

- 最下层的叶子一定集中在左部的连续位置.

- 倒数二层,若有叶子节点,则一定在右部的连续位置/

- 如果结点度为1,则该结点只有有孩子,即不存在只有右子树的情况.

- 同样结点书的二叉树,完全二叉树的深度最小.

