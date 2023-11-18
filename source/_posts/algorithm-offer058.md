---
title: 剑指 Offer II 058. 我的日程安排表
tags: [LeetCode, 二叉搜索树]
categories: [剑指 Offer II]
index_img: https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/algorithm058.png   # 封面图
# banner_img: /img/post_banner.jpg  # 文章顶部大图
---

## 题目要求

[leetcode地址](https://leetcode.cn/problems/fi9suh/?envType=study-plan-v2&envId=coding-interviews-special)

请实现一个 *MyCalendar* 类来存放你的日程安排。如果要添加的时间内没有其他安排，则可以存储这个新的日程安排。

*MyCalendar* 有一个 book(int start, int end)方法。它意味着在 start 到 end 时间内增加一个日程安排，注意，这里的时间是半开区间，即 [start, end), 实数 x 的范围为，  start <= x < end。

当两个日程安排有一些时间上的交叉时（例如两个日程安排都在同一时间内），就会产生重复预订。

每次调用 MyCalendar.book方法时，如果可以将日程安排成功添加到日历中而不会导致重复预订，返回 true。否则，返回 false 并且不要将该日程安排添加到日历中。

请按照以下步骤调用 *MyCalendar* 类: MyCalendar cal = new MyCalendar(); MyCalendar.book(start, end)

#### 示例

**输入:**
["MyCalendar","book","book","book"]
[[],[10,20],[15,25],[20,30]]
**输出:**
[null,true,false,true]
**解释:**
MyCalendar myCalendar = new MyCalendar();
MyCalendar.book(10, 20); // returns true
MyCalendar.book(15, 25); // returns false ，第二个日程安排不能添加到日历中，因为时间 15 已经被第一个日程安排预定了
MyCalendar.book(20, 30); // returns true ，第三个日程安排可以添加到日历中，因为第一个日程安排并不包含时间 20

#### 提示

- 每个测试用例，调用 MyCalendar.book 函数最多不超过 1000次。
- 0 <= start < end <= 109

## 思路及步骤

根据题意我们可以知道，日程安排不能出现时间冲突，即时间区间按时序放入且不交叉。因为最近做的二叉搜索树的题比较多，因此很容易想到了它的特性，比当前节点小的元素只能在左边，比当前节点大的元素只能在右边。本题中，换成时间区间同样适用。

步骤如下：

- 首先构建一个二叉搜索树TreeNode
TreeNode 类表示 BST 中的节点。每个节点包含对其左右子节点的引用，以及表示预订事件的开始和结束时间的 start 和 end 字段。有两个构造函数可用：一个使用给定的 start 和 end 值初始化节点，另一个是默认构造函数。
- book 方法用于在日历中预订事件。它接受一个 start 和 end 参数，表示事件的开始和结束时间。该方法返回一个布尔值，指示预订是否成功。

该方法按照以下步骤执行：

1. 如果 root 节点为空，则说明日历为空。在这种情况下，创建一个新的具有给定 start 和 end 值的 TreeNode 并将其分配为 root。方法返回 true，表示预订成功。
2. 如果 root 节点不为空，初始化指针 p 指向 root。
3. 当指针 p 不为空时，重复以下步骤：
   - 如果新事件的 end 值小于 p 的 start 值（即新事件结束在现有事件开始之前），则将新事件插入到 p 的左子树中。
        - 如果 p 的左子节点为空，创建一个具有给定 start 和 end 值的新 TreeNode 并将其分配为 p 的左子节点。方法返回 true，表示预订成功。
        - 如果 p 的左子节点不为空，则更新指针 p 指向其左子节点，并继续循环。
   - 如果新事件的 start 值大于等于 p 的 end 值（即新事件开始在现有事件结束之后），则将新事件插入到 p 的右子树中。
        - 如果 p 的右子节点为空，创建一个具有给定 start 和 end 值的新 TreeNode 并将其分配为 p 的右子节点。方法返回 true，表示预订成功。
        - 如果 p 的右子节点不为空，则更新指针 p 指向其右子节点，并继续循环。
   - 如果以上条件都不满足，则说明新事件与现有事件存在重叠。在这种情况下，方法返回 false，表示预订失败。

如果循环完成而没有找到合适的位置来插入新事件，则方法返回 false，表示预订失败。

## 代码实现（java）

```java
class MyCalendar {

    // TreeNode类表示BST中的节点
    class TreeNode {
        TreeNode left;
        TreeNode right;
        int start;
        int end;
        
        public TreeNode(int start, int end) {
            this.start = start;
            this.end = end;
        }

        public TreeNode() {

        }
    }

    TreeNode root; // BST的根节点

    public MyCalendar() {

    }
    
    // 方法用于在日历中预订事件
    public boolean book(int start, int end) {
        if(root == null) {
            // 如果日历为空，创建根节点并返回true
            root = new TreeNode(start,end);
            return true;
        }
        TreeNode p = root;
        while(p!= null) {
            if(end <= p.start) {
                // 如果新事件的结束时间早于现有事件的开始时间，
                // 则将其插入到左子树中
                if(p.left == null) {
                    p.left = new TreeNode(start, end);
                    return true;
                }
                p = p.left;
            } else if(start >= p.end) {
                // 如果新事件的开始时间晚于等于现有事件的结束时间，
                // 则将其插入到右子树中
                if (p.right == null) {
                    p.right = new TreeNode(start,end);
                    return true;
                }
                p = p.right;
            } else {
                // 如果新事件与现有事件存在重叠，
                // 返回false表示预订失败
                return false;
            }
        }
        return false;
    }
}
```

## 代码实现（golang）

```go
type MyCalendar struct {
    root *MyTreeNode
}

type MyTreeNode struct {
    left  *MyTreeNode
    right *MyTreeNode
    start int
    end   int
}

func Constructor() MyCalendar {
    return MyCalendar{}
}

func (this *MyCalendar) Book(start int, end int) bool {
    if this.root == nil {
        // If the calendar is empty, create root node and return true
        this.root = &MyTreeNode{start: start, end: end}
        return true
    }

    p := this.root
    for p != nil {
        if end <= p.start {
        // If the new event ends before existing event starts,
        // insert it in the left subtree
            if p.left == nil {
                p.left = &MyTreeNode{start: start, end: end}
                return true
            }
            p = p.left
        } else if start >= p.end {
            // If the new event starts after existing event ends,
            // insert it in the right subtree
            if p.right == nil {
                p.right = &MyTreeNode{start: start, end: end}
                return true
            }
            p = p.right
        } else {
            // If there is an overlap between new and existing events,
            // return false
            return false
        }
    }
    return false
}
```
