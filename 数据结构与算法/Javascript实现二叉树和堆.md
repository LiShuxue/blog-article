## 树的概念

树属于非线性结构，是由结点和边组成的且不存在着任何环的一种数据结构。没有结点的树称为空树。

## 树的常用术语

- 节点的度：某个节点子树的个数。
- 树的度：树中各节点的度的最大值。
- 叶子节点：度为 0 的节点。
- 父节点：度不为 0 的节点。
- 子节点：某节点的后继节点。
- 兄弟节点：同一父节点的各节点彼此是兄弟节点。
- 路径和路径长度：路径指从一个节点到另一个节点。路径长度指路径所包含的边的个数。
- 节点的层次：根节点是第一层。以此类推。
- 节点的深度：节点所在的层次。
- 树的深度：树中节点最大的层次。
- 节点的高度：从该节点到叶子节点的最长路径长。
- 树的高度：为根的高度。
- 有序树：树中任意节点的子节点之间有顺序关系，这种树称为有序树。
- 无序树：树中任意节点的子节点之间没有顺序关系，这种树称为无序树，也称为自由树。

## 二叉树

树中的每一个节点最多只能由两个子节点，这样的树就称为二叉树。由左子树 TL 和右子树 TR 两个不相交的二叉树组成。

二叉树可以为空，也就是没有节点。也可以只有根节点或者只有左子树，右子树。

### 完全二叉树

除了二叉树最后一层外，其他各层的节点数都达到了最大值。

最后一层的叶子节点从左向右是连续存在，只缺失右侧若干叶子节点。

最左边只有左节点没有右节点，中间既有左又有右，这不是完全二叉树。

### 满二叉树

所有父结点都存在左子树和右子树，并且所有叶子节点都在同一层上，这样的二叉树称为满二叉树。

### 二叉搜索树 BST（二叉排序树，排序二叉树）

节点的左子树小于该节点本身，右子树大于该节点，每个节点都符合这样的规则，对二叉搜索树进行中序遍历就得得到一个有序的序列。

二叉搜索树比普通树查找更快，查找、插入、删除的时间复杂度为 O（logN）。但是二叉查找树有一种极端的情况，就是会变成一种线性链表似的结构。此时时间复杂度就变成了 O（N），为了解决这种情况，出现了二叉平衡树。

### 平衡二叉树

任意节点的子树的高度差都小于等于 1。

### 平衡二叉搜索树，也叫 AVL 树

AVL 树也规定了左结点小于根节点，右结点大于根节点。并且还规定了左子树和右子树的高度差不得超过 1。

他是自平衡的，由于要维持自身的平衡，所以进行插入和删除结点操作的时候，需要对结点进行频繁的旋转。

### 红黑树 RBT

红黑树是 AVL 树的变体，也需要在进行插入和删除操作时通过特定操作保持二叉查找树的平衡。但它的左右子树高差有可能大于 1。所以红黑树不是严格意义上的平衡二叉树。

- 性质 1. 结点是红色或黑色。
- 性质 2. 根结点是黑色。
- 性质 3. 所有叶子都是黑色，并且是空节点。
- 性质 4. 每个红色结点的两个子结点都是黑色。
- 性质 5. 从任一节结点其每个叶子的所有路径都包含相同数目的黑色结点。

为了保持红黑树的性质，我们可以对相关结点做一系列的调整，通过对树进行旋转（例如左旋和右旋操作），即修改树中某些结点的颜色及指针结构，以达到对红黑树进行插入、删除结点等操作时，红黑树依然能保持它特有的性质（五点性质）。

### B 树（B-树）

B 树，有的叫 B-树，（B-树其实读法不对）。也是一种平衡树，但不一定是二叉树，是多路自平衡的搜索树。我们常说 m 阶的 B 树，表示的就是他的节点最多有 m 个子节点。

### B+树

B 树的优化。

## 堆

堆是一种特殊的完全二叉树，可以用数组存储。假设每一个节点的编号为 i，那么它的左子节点编号是 2i，而右子节点是 2i+1。

### 大根堆

所有父结点都比子结点大的完全二叉树。

### 小根堆

所有父结点都比子结点小的完全二叉树。

## 树的遍历

- 先序遍历：先访问根节点，再访问左子树，最后访问右子树。
- 中序遍历：先左子树，再根节点，最后右子树。
- 后序遍历：先左子树，再右子树，最后根节点。
- 层序遍历：每一层从左到右访问每一个节点。一般很少使用。

前序、中序、后序遍历是对二叉树进行深度优先遍历的几种方式。所以需要递归，每一个子树遍历时依然按照此时的遍历顺序。

如下图：
![二叉树](https://cdn.lishuxue.site/blog/image/数据结构与算法/sorttree.png)

先序遍历：FCADBEHGM  
中序遍历：ACBDFHEMG  
后序遍历：ABDCHMGEF  
层序遍历：FCEADHGBM，一般很少使用。

## 二叉树的存储

二叉树存储方式可以是数组或者链表。

### 使用数组

完全二叉树可以直接按从上到下，从左到右的方式存储数据。非完全二叉树需要转换成完全二叉树才能按照上面的方案存储，但这样会浪费很大的存储空间。

### 使用链表

二叉树最常见的存储方式为链表，每一个节点封装成一个 Node，Node 中包含存储的数据、左节点的引用和右节点的引用。

## js 实现二叉搜索树

### 操作

- insert(key) 向树中插入一个新的键。
- search(key) 在树中查找一个键，如果节点存在，则返回 true；如果不存在，则返回 false。
- preOrderTraverse 通过先序遍历方式遍历所有节点。
- inOrderTraverse 通过中序遍历方式遍历所有节点。
- postOrderTraverse 通过后序遍历方式遍历所有节点。
- min 返回树中最小的值/键。
- max 返回树中最大的值/键。
- remove(key) 从树中移除某个键。

### js 实现

```js
class Node {
  constructor(key) {
    this.left = null;
    this.right = null;
    this.key = key;
  }
}

// 二叉搜索树（特点：左子树节点值 < 根节点，右子树节点值 > 根节点）
class BinarySearchTree {
  constructor() {
    this.root = null;
  }

  insert(key) {
    const newNode = new Node(key);
    if (this.root === null) {
      this.root = newNode;
    } else {
      this.insertNode(this.root, newNode);
    }
  }

  insertNode(root, node) {
    if (node.key < root.key) {
      // 左侧插入
      if (root.left === null) {
        root.left = node;
      } else {
        this.insertNode(root.left, node); // 递归查找
      }
    } else {
      // 右侧插入
      if (root.right === null) {
        root.right = node;
      } else {
        this.insertNode(root.right, node); // 递归查找
      }
    }
  }

  preOrderTraverse() {
    let result = [];
    this.preOrderTraverseNode(this.root, result);
    return result;
  }

  preOrderTraverseNode(node, result) {
    if (node === null) return result;
    result.push(node.key);
    this.preOrderTraverseNode(node.left, result);
    this.preOrderTraverseNode(node.right, result);
  }

  inOrderTraverse() {
    let result = [];
    this.inOrderTraverseNode(this.root, result);
    return result;
  }

  inOrderTraverseNode(node, result) {
    if (node === null) return result;
    this.inOrderTraverseNode(node.left, result);
    result.push(node.key);
    this.inOrderTraverseNode(node.right, result);
  }

  postOrderTraverse() {
    let result = [];
    this.postOrderTraverseNode(this.root, result);
    return result;
  }

  postOrderTraverseNode(node, result) {
    if (node === null) return result;
    this.postOrderTraverseNode(node.left, result);
    this.postOrderTraverseNode(node.right, result);
    result.push(node.key);
  }

  // 最小值永远在最左边
  min() {
    if (!this.root) return null;
    let node = this.root;
    while (node.left !== null) {
      node = node.left;
    }
    return node.key;
  }

  // 最大值永远在最右边
  max() {
    if (!this.root) return null;
    let node = this.root;
    while (node.right !== null) {
      node = node.right;
    }
    return node.key;
  }

  search(key) {
    let node = this.root;
    while (node !== null) {
      if (key < node.key) {
        node = node.left;
      } else if (key > node.key) {
        node = node.right;
      } else {
        return true;
      }
    }
    return false;
  }
}
```
