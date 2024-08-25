---
layout: post
title: 二叉树遍历
date: 2024-08-25 17:30:00 +0800
categories: algorithm
tags: binary-tree
published: true
---

* content
{:toc}

## 二叉树

```kotlin
class TreeNode(var value: Int) {
    var left: TreeNode? = null
    var right: TreeNode? = null
}
```

* **父亲（parent node）**：对于除根以外的每个结点，定义为从该结点到根路径上的第二个结点。根结点没有父结点。
* **祖先（ancestor）**：一个结点到根结点的路径上，除了它本身外的结点。根结点的祖先集合为空。
* **子结点（child node）**：如果 u 是 v 的父亲，那么 v 是 u 的子结点。子结点的顺序一般不加以区分，二叉树是一个例外。
* **结点的深度（depth）**：到根结点的路径上的边数。
* **树的高度（height）**：所有结点的深度的最大值。
* **兄弟（sibling）**：同一个父亲的多个子结点互为兄弟。
* **后代（descendant）**：子结点和子结点的后代。或者理解成：如果 u 是 v 的祖先，那么 v 是 u 的后代。

### 完整二叉树

完整二叉树（full/proper binary tree）：每个结点的子结点数量均为 0 或者 2 的二叉树。换言之，每个结点或者是树叶，或者左右子树均非空。

![secret-box]({{ '/styles/images/binary-tree/tree-binary-proper.svg' | prepend: site.baseurl }}){:width="208" height="auto"} 

### 完全二叉树

完全二叉树（complete binary tree）：只有最下面两层结点的度数可以小于 2，且最下面一层的结点都集中在该层最左边的连续位置上。

![secret-box]({{ '/styles/images/binary-tree/tree-binary-complete.svg' | prepend: site.baseurl }}){:width="208" height="auto"} 

### 完美二叉树

完美二叉树（perfect binary tree）：所有叶结点的深度均相同，且所有非叶节点的子节点数量均为 2 的二叉树称为完美二叉树。有时也称满二叉树。

![secret-box]({{ '/styles/images/binary-tree/tree-binary-perfect.svg' | prepend: site.baseurl }}){:width="208" height="auto"} 

### 遍历

* 左子结点一定是在右子结点前面
* 前中后指的是父结点相对子结点的位置

假设父结点为 p，左子结点为 l，右子结点为 r：

* 前序遍历为 plr，父结点位于左右子结点之前
* 中序遍历为 lpr，父结点位于左右子结点之中
* 后序遍历为 lrp，父结点位于左右子结点之后

由于子结点也是一棵二叉树，以同样的顺序递归左右子树即可。

## 递归遍历

### 前序

```kotlin
val list = mutableListOf<Int>()
fun preorderTraversal(root: TreeNode?): List<Int> {
    if (root != null) {
        list.add(root.value)
        preorderTraversal(root.left)
        preorderTraversal(root.right)
    }
    return list
}
```

### 中序

```kotlin
val list = mutableListOf<Int>()
fun inorderTraversal(root: TreeNode?): List<Int> {
    if (root != null) {
        inorderTraversal(root.left)
        list.add(root.value)
        inorderTraversal(root.right)
    }
    return list
}
```

### 后序

```kotlin
val list = mutableListOf<Int>()
fun postorderTraversal(root: TreeNode?): List<Int> {
    if (root != null) {
        postorderTraversal(root.left)
        postorderTraversal(root.right)
        list.add(root.value)
    }
    return list
}
```

可以看到无论是前序、中序还是后序，使用递归都很简单，书写形式基本一致，只是调换访问父结点的顺序。

## 非递归遍历

### 前序

```kotlin
fun preorderTraversal(root: TreeNode?): List<Int> {
    val list = mutableListOf<Int>()
    val stack = Stack<TreeNode>()
    stack.push(root)
    while (stack.isNotEmpty()) {
        val node = stack.pop()
        if (node != null) {
            list.add(node.value)
            node.right?.apply(stack::push)
            node.left?.apply(stack::push)
        }
    }
    return list
}
```

### 中序

```kotlin
fun inorderTraversal(root: TreeNode?): List<Int> {
    val list = mutableListOf<Int>()
    val stack = Stack<TreeNode>()
    var node: TreeNode? = root
    while (node != null || stack.isNotEmpty()) {
        if (node != null) {
            stack.push(node)
            node = node.left
        } else {
            node = stack.pop()
            list.add(node.value)
            node = node.right
        }
    }
    return list
}
```

### 后序

**1个栈**
```kotlin
fun postorderTraversal(root: TreeNode?): List<Int> {
    val list = mutableListOf<Int>()
    val stack = Stack<TreeNode>()
    var node: TreeNode? = root
    var pre: TreeNode? = null   // 标记最近访问过的结点
    while (node != null || stack.isNotEmpty()) {
        if (node != null) {
            stack.push(node)
            node = node.left
        } else {
            node = stack.pop()
            if (node.right != null && node.right != pre) {
                stack.push(node)
                node = node.right
            } else {
                list.add(node.value)
                pre = node
                node = null
            }
        }
    }
    return list
}
```

**2个栈**
```kotlin
fun postorderTraversal(root: TreeNode?): List<Int> {
    val list = mutableListOf<Int>()
    if (root == null) return list
    val stack1 = Stack<TreeNode>()
    val stack2 = Stack<Int>()
    stack1.push(root)
    while (stack1.isNotEmpty()) {
        val node = stack1.pop()
        stack2.push(node.value)
        node.left?.apply(stack1::push)
        node.right?.apply(stack1::push)
    }
    while (stack2.isNotEmpty()) list.add(stack2.pop())
    return list
}
```

最后，附上 LeetCode 相关题目：

* [LeetCode 144. Binary Tree Preorder Traversal](https://leetcode.com/problems/binary-tree-preorder-traversal){:target="_blank"}
* [LeetCode 094. Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal){:target="_blank"}
* [LeetCode 145. Binary Tree Postorder Traversal](https://leetcode.com/problems/binary-tree-postorder-traversal){:target="_blank"}
* [LeetCode 590. N-ary Tree Postorder Traversal](https://leetcode.com/problems/n-ary-tree-postorder-traversal){:target="_blank"}

* [LeetCode 105. Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal){:target="_blank"}
* [LeetCode 106. Construct Binary Tree from Inorder and Postorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal){:target="_blank"}
* [LeetCode 889. Construct Binary Tree from Preorder and Postorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-postorder-traversal){:target="_blank"}

<!-- https://oi-wiki.org/graph/tree-basic/#%E4%BA%8C%E5%8F%89%E6%A0%91 -->
<!-- https://blog.csdn.net/Mr_liubai/article/details/120569313 -->
<!-- https://leetcode.com/problems/binary-tree-preorder-traversal/ -->
<!-- https://leetcode.com/problems/binary-tree-inorder-traversal/ -->
<!-- https://leetcode.com/problems/binary-tree-postorder-traversal/ -->
<!-- https://leetcode.com/problems/n-ary-tree-postorder-traversal/ -->

<!-- https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/ -->
<!-- https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/ -->
<!-- https://leetcode.com/problems/construct-binary-tree-from-preorder-and-postorder-traversal/ -->