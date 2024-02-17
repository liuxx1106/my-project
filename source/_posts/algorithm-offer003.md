---
date: 2023-07-18 21:30:21
title: 剑指 Offer II 003. 前 n 个数字二进制中 1 的个数
tags: [LeetCode, 奇偶性质]
categories: [剑指 Offer II]
index_img: https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/algorithm3.jpg    # 封面图
# banner_img: /img/post_banner.jpg  # 文章顶部大图
---

## 题目要求

[leetcode地址](https://leetcode.cn/problems/w3tCBm/?envType=study-plan-v2&envId=coding-interviews-special)
给定一个非负整数 n ，请计算 0 到 n 之间的每个数字的二进制表示中 1 的个数，并输出一个数组。

#### 示例1
>
> 输入: n = 2
输出: [0,1,1]
解释:
0 --> 0
1 --> 1
2 --> 10

#### 示例2
>
> 输入: n = 5
输出: [0,1,1,2,1,2]
解释:
0 --> 0
1 --> 1
2 --> 10
3 --> 11
4 --> 100
5 --> 101

说明 :

- 0 <= n <= 105

进阶:

- 给出时间复杂度为 O(n*sizeof(integer)) 的解答非常容易。但你可以在线性时间 O(n) 内用一趟扫描做到吗？
- 要求算法的空间复杂度为 O(n) 。
- 你能进一步完善解法吗？要求在C++或任何其他语言中不使用任何内置函数（如 C++ 中的 __builtin_popcount ）来执行此操作。

## 思路

求二进制中的1的个数，我们可以根据数的奇偶性原则找到对应关系：

- 当数字i为偶数时，它的二进制中1的个数等于i/2二进制中1的个数。因为在二进制中除以2即相当于把原来的数字右移一位，在高位补0，如 110（6）右移一位为011（3）。由此可得到状态方程：f(i) = f(i>>1), i为偶数；
- 当数字i为奇数时，它的二进制中1的个数等于它的前一位数字中1的个数加1（因为它的前一位数字为偶数，最低位为0），可得到状态方程f(i) = f(i-1) + 1;由此可得到以下代码：

## 代码实现（java）

```java
class Solution {
    //利用奇偶性质，偶数的二进制1的个数一定等于右移一位后的1的个数
    //奇数的二进制1的个数一定等于前一个数的1的个数+1
    //得到状态方程：f(i) = f(i-1) +1; i为奇数
    // f(i) = f(i>>1)
    public int[] countBits(int n) {
        int[] ans = new int[n+1];
        for(int i=1;i<=n;i++) {
            ans[i] = ans[i>>1] + (i&1);
        }
        return ans;

    }
} 
```

## 代码实现（golang）

```go
func countBits(n int) []int {
    //利用奇偶性质，偶数的二进制1的个数一定等于右移一位后的1的个数
    //奇数的二进制1的个数一定等于前一个数的1的个数+1
    //得到状态方程：f(i) = f(i-1) +1; i为奇数
    // f(i) = f(i>>1)
    ans := make([]int, n+1)
    for i := 1;i<=n; i++ {
        if((i&1) == 0) { // 偶数
            ans[i] = ans[i>>1]
        }else {
            ans[i] = ans[i-1] +1
        }
    }
    return ans
}
```
