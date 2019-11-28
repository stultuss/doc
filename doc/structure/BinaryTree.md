二叉树
=========================

> 记录面试中的一些问题
>

## 代码实现

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

module.exports = BST;
```

