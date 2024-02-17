---
date: 2023-09-20 21:00:00
title: 剑指 Offer II 056. 两数之和 IV - 输入二叉搜索树
tags: [LeetCode, 二叉搜索树]
categories: [剑指 Offer II]
index_img: https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/algorithm056.png   # 封面图
# banner_img: /img/post_banner.jpg  # 文章顶部大图
---

## 题目要求

[leetcode地址](https://leetcode.cn/problems/opLdQZ/?envType=study-plan-v2&envId=coding-interviews-special)
给定一个二叉搜索树的 根节点 root 和一个整数 k , 请判断该二叉搜索树中是否存在两个节点它们的值之和等于 k 。假设二叉搜索树中节点的值均唯一。

#### 示例1

> 输入: root = [8,6,10,5,7,9,11], k = 12
> 输出: true
解释:
节点 5 和节点 7 之和等于 12

#### 示例2

> 输入: root = [8,6,10,5,7,9,11], k = 22
> 输出: false
解释:
不存在两个节点值之和为 22 的节点

#### 提示

- 二叉树的节点个数的范围是  [1, 104].
- -104 <= Node.val <= 104
- root 为二叉搜索树
- -105 <= k <= 105

## 思路

根据题意和示例，我们可以比较清晰地找到剪枝的条件有两个：

解法一：dfs + 哈希表

> - 哈希表用来存放每次遍历的节点
> - 对二叉树进行遍历，每次判断hash表中是否存在k-root.val,若存在则返回true;否则存入hash表，直到搜索完整棵树

解法二：dfs + 数组 + 双指针

> - 利用二叉搜索树中序遍历为递增数组的特性，先中序遍历得到递增数组orderNums
> - 定义左右指针，若orderNums[left] + orderNums[right] == k,则返回true;
> 若小于则left++;若大于则right--;当left不小于right时结束；

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
    // 解法一：
    boolean flag = false;
    Set<Integer> set = new HashSet<>();
    public boolean findTarget(TreeNode root, int k) {
        dfs(root, k);
        return flag;
    }

    private void dfs(TreeNode root, int k) {
        if(root == null) {
            return;
        }
        dfs(root.left,k);
        if(set.contains(k-root.val)) {
            flag = true;
            return;
        } else {
            set.add(root.val);
        }
        dfs(root.right,k);
    }
    // 解法二：
    List<Integer> orderNums = new ArrayList<>();
    public boolean findTarget(TreeNode root, int k) {
        dfs(root);
        int left = 0,right = orderNums.size() - 1;
        while(left < right) {
            if(orderNums.get(left) + orderNums.get(right) == k) {
                return true;
            }
            if(orderNums.get(left) + orderNums.get(right) < k) {
                left++;
            } else {
                right--;
            }
        }
        return false;
    }

    void dfs(TreeNode root) {
        if(root == null) {
            return;
        }
        dfs(root.left);
        orderNums.add(root.val);
        dfs(root.right);
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
    // 解法一：
    func findTarget(root *TreeNode, k int) bool {
        set := map[int]struct{}{}
        var dfs func(*TreeNode) bool
        dfs = func(node *TreeNode) bool {
        if node == nil {
            return false
        }
        if _, ok := set[k-node.Val]; ok {
            return true
        }
        set[node.Val] = struct{}{}
        return dfs(node.Left) || dfs(node.Right)
        }
        return dfs(root)
    }
    // 解法二：
    func findTarget(root *TreeNode, k int) bool {
    arr := []int{}
    var inorderTraversal func(*TreeNode)
    inorderTraversal = func(node *TreeNode) {
        if node != nil {
            inorderTraversal(node.Left)
            arr = append(arr, node.Val)
            inorderTraversal(node.Right)
        }
    }
    inorderTraversal(root)

    left, right := 0, len(arr)-1
    for left < right {
        sum := arr[left] + arr[right]
        if sum == k {
            return true
        }
        if sum < k {
            left++
        } else {
            right--
        }
    }
    return false
    }
```
