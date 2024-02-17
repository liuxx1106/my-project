---
date: 2023-07-25 21:00:00
title: 剑指 Offer II 009. 乘积小于 K 的连续子数组的个数
tags: [LeetCode, 双指针]
categories: [剑指 Offer II]
index_img: https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/algorithm5.jpg    # 封面图
# banner_img: /img/post_banner.jpg  # 文章顶部大图
---

## 题目要求

[leetcode地址](https://leetcode.cn/problems/ZVAVXX/?envType=study-plan-v2&envId=coding-interviews-special)
给定一个正整数数组 nums和整数 k ，请找出该数组内乘积小于 k 的连续的子数组的个数。

#### 示例1
>
> 输入: nums = [10,5,2,6], k = 100
输出: 8
解释: 8 个乘积小于 100 的子数组分别为: [10], [5], [2], [6], [10,5], [5,2], [2,6], [5,2,6]。
需要注意的是 [10,5,2] 并不是乘积小于100的子数组。

#### 示例2
>
> 输入: nums = [1,2,3], k = 0
输出: 0

#### 提示

1 <= nums.length <= 3 * 10 ^ 4
1 <= nums[i] <= 1000
0 <= k <= 10 ^ 6

## 思路

- 1. 双指针遍历。定义左右指针left和right, right向右扫描，找到以left为起点的符合条件的子数组；当不符合条件时left右移，同时重置right，重复之前的操作；
- 2. 我们遍历整个数组 nums，定义两个指针 left 和 right，表示子数组的左右端点，用 res 变量保存符合条件的连续子数组的个数，并且定义 product 变量，表示当前子数组的乘积。
遍历数组中的每一个数，把当前数的值累乘到 product 中，如果 product 大于等于设定的值 k，向左移动左指针 left 并把左端元素除以 product 的值，直到当前区间内所有数的乘积都小于 k 为止。此时，减去左右端点之间的数的个数加一（因为是左开右闭区间），得到符合条件的区间的个数。
最后返回 res 变量的值，即为符合条件的连续子数组的个数。

## 代码实现（java）

```java
class Solution {
 方法1. 双指针
 public static int numSubarrayProductLessThanK(int[] nums, int k) {
    int count = 0; // 记录个数
        int left = 0; //左边界
        while(left< nums.length) {
            int sum = 1; // 乘积
            int right = left; // 每次重置右边界
            while(right < nums.length) {
                sum *= nums[right]; // 累乘判断是否小于K
                if(sum < k) {
                    count++;
                    right++;
                } else {
                    break;
                }
            }
            left++; // 左边界前进一位
        }
        return count;
     }

 方法2. 滑动窗口
 public static int numSubarrayProductLessThanK(int[] nums, int k) {
    if (k <= 1) return 0;
        int left = 0;
        int right = 0;
        int res = 0;
        int product = 1;
        while (right < nums.length) {
            product *= nums[right];
            while (left <= right && product >= k) {
                product /= nums[left++];
            }
            //
            res += (right - left + 1);
            right++;
        }
        return res;
    }
}
```

## 代码实现（golang）

```go
//1.map解法
方法1. 双指针
    func numSubarrayProductLessThanK(nums []int, k int) int {
    count := 0 // 记录个数
    left := 0  // 左边界
    for left < len(nums) {
        sum := 1     // 乘积
        right := left // 每次重置右边界
        for right < len(nums) {
            sum *= nums[right] // 累乘判断是否小于K
            if sum < k {
                count++
                right++
            } else {
                break
            }
        }
        left++ // 左边界前进一位
    }
    return count
}

 方法2. 滑动窗口
    func numSubarrayProductLessThanK(nums []int, k int) int {
    if k <= 1 {
        return 0
    }
    left, right, res, product := 0, 0, 0, 1

    for right < len(nums) {
        product *= nums[right]
        for left <= right && product >= k {
            product /= nums[left]
            left++
        }
        res += (right - left + 1)
        right++
    }
     
    return res
}
```
