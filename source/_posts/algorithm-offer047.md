---
date: 2023-08-03 21:00:00
title: 剑指 Offer II 047. 二叉树剪枝
tags: [LeetCode, 二叉树, 深度优先搜索]
categories: [剑指 Offer II]
index_img: https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/algorithm7.jpg   # 封面图
# banner_img: /img/post_banner.jpg  # 文章顶部大图
---

## 题目要求

[leetcode地址](https://leetcode.cn/problems/pOCWxh/description/?envType=study-plan-v2&envId=coding-interviews-special)
给定一个二叉树 根节点 root ，树的每个节点的值要么是 0，要么是 1。请剪除该二叉树中所有节点的值为 0 的子树。
节点 node 的子树为 node 本身，以及所有 node 的后代。

#### 示例1

> 输入: [1,null,0,0,1]
> 输出: [1,null,0,null,1]
解释:
只有红色节点满足条件“所有不包含 1 的子树”。
右图为返回的答案。
![示例1](https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/offer7-1.png)

#### 示例2

> 输入: [1,0,1,0,0,0,1]
> 输出: [1,null,1,null,1]
解释:
![示例2](https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/offer7-2.png)

#### 示例3

> 输入: [1,1,0,1,1,0,1,0]
> 输出: [1,1,0,1,1,null,1]
解释:
![示例3](https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/offer7-3.png)

#### 提示

- 二叉树的节点个数的范围是 [1,200]
- 二叉树节点的值只会是 0 或 1

## 思路

根据题意和示例，我们可以比较清晰地找到剪枝的条件有两个：

- 1. node.val == 0;
- 2. node.left == null && node.right == null

因此，我们可以利用深度优先搜索遍历节点的子节点，递归，即可得到最终答案。

## 代码实现（java）

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public TreeNode pruneTree(TreeNode root) {
        return dfs(root);
    }

    TreeNode dfs(TreeNode node) {
        if(node == null) {
            return node;
        }
        node.left = dfs(node.left);
        node.right = dfs(node.right);
        if(node.val == 0 && node.left == null && node.right == null) {
            node = null;
        }
        return node;
    }
}
```

## 代码实现（golang）

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func pruneTree(root *TreeNode) *TreeNode {
    if root == nil {
        return root
    }
    root.Left = pruneTree(root.Left)
    root.Right = pruneTree(root.Right)
    if root.Val == 0 && root.Left == nil && root.Right == nil {
        root = nil
    }
    return root
}
```
