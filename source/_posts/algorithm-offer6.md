---
title: 剑指 Offer II 046. 二叉树的右侧视图
tags: [LeetCode, 二叉树, 深度优先搜索]
categories: [剑指 Offer II]
index_img: https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/algorithm6.jpg    # 封面图
# banner_img: /img/post_banner.jpg  # 文章顶部大图
---

## 题目要求

[leetcode地址](https://leetcode.cn/problems/WNC0Lk/description/?envType=study-plan-v2&envId=coding-interviews-special)
给定一个二叉树的 根节点 root，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。

#### 示例1

![示例1](https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/offer6-1.png)

> 输入: [1,2,3,null,5,null,4]
> 输出: [1,3,4]

#### 示例2

> 输入: [1,null,3]
> 输出: [1,3]

#### 示例3

> 输入: []
> 输出: []

#### 提示

- 二叉树的节点个数的范围是 [0,100]
- -100 <= Node.val <= 100

## 思路

- `DFS`深度优先搜索，看到该题目，我们的首先想到的应该是dfs, 步骤：
  - 从根节点一直沿着右侧节点遍历，最右侧节点全部为待返回的值，直接加入结果即可，记录层高deep；
  - 遍历左边节点，如果curDeep > deep,且node.right有值，则为右侧元素加入结果；
  - 递归；

![深度优先搜索图解](https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/offer6-2.png)

执行过程：

（1） 第一轮右侧直接遍历到最底，找到1，3，6，此时deep等于2（从0开始的）
（2） 第二轮从2开始遍历，由于2、4，层高均不大于deep,跳过
（3） 第三轮从5开始，找到8，此时deep=3
（4） 第四轮找到9,结束。结果为1，3，6，8，9

- `BFS`没电了，待补充, 步骤：
  - ；
  - ；
  - ；

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
    List<Integer> res = new ArrayList<>();
    int deep;
    public List<Integer> rightSideView(TreeNode root) {
        deep = -1;
        dfs(root,0);
        return res;

    }
    void dfs(TreeNode node, int curDeep) {
        if(node == null) {
            return;
        }
        if(curDeep > deep) {
            deep = curDeep;
            res.add(node.val);
        }
        dfs(node.right,curDeep + 1);
        dfs(node.left, curDeep + 1);
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
func rightSideView(root *TreeNode) []int {
    res := []int{}
    var deep int = -1
    dfs(root, 0, &deep, &res)
    return res
}

func dfs(node *TreeNode, curDeep int, deep *int, res *[]int) {
    if node == nil {
        return
    }
    if curDeep > *deep {
        *deep = curDeep
        *res = append(*res, node.Val)
    }
    dfs(node.Right, curDeep+1, deep, res)
    dfs(node.Left, curDeep+1, deep, res)
}
```
