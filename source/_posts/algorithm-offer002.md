---
date: 2023-07-15 21:31:11
title: 剑指 Offer II 002. 二进制加法
tags: [LeetCode, 二进制加法]
categories: [剑指 Offer II]
index_img: https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/algorithm2.jpg    # 封面图
# banner_img: /img/post_banner.jpg  # 文章顶部大图
---

## 题目要求

[leetcode地址](https://leetcode.cn/problems/JFETK5/description/?envType=study-plan-v2&envId=coding-interviews-special)

* 剑指 Offer II 002. 二进制加法
给定两个 01 字符串 a 和 b ，请计算它们的和，并以二进制字符串的形式输出。
输入为 非空 字符串且只包含数字 1 和 0。

## 注意

示例 1:

> 输入: a = "11", b = "10"
输出: "101"

示例 2:

> 输入: a = "1010", b = "1011"
输出: "10101"

提示：

每个字符串仅由字符 '0' 或 '1' 组成。
1 <= a.length, b.length <= 10^4
字符串如果不是 "0" ，就都不含前导零。

* 提示:
* -2 ^ 31 <= a, b <= 2 ^ 31 - 1
* b != 0

## 思路

这道题还是比较好理解的，类似整数加法，只不过是“逢2进1”，我们只需要以两个数的右边对齐，逐位开始相加（需加上进位，初始值位0），若大于2则进位1，取2的余数作为结果拼接上；若小于2直接拼接到结果。两个字符串都遍历完且没有进位返回结果。

## 注意事项

- 本题给出的二进制数字是字符串形式，不可以转化成 int 型，因为可能溢出；
* 两个「加数」的字符串长度可能不同；
* 在最后，如果进位 carry 不为 0，那么最后需要计算进位；

## 代码实现（java）

```java
class Solution {
    public String addBinary(String a, String b) {
       StringBuilder res = new StringBuilder(); // 返回结果
        int i = a.length() - 1; //遍历索引
        int j = b.length() - 1; //遍历索引
        int carry = 0; // 进位初始值为0
        while (i >= 0 || j >= 0 || carry != 0) { // 当字符串a,b都遍历完且carry为0时结束。
            int digitA = i >= 0 ? a.charAt(i) - '0' : 0; // 从右边开始取每一位的值
            int digitB = j >= 0 ? b.charAt(j) - '0' : 0;
            int sum = digitA + digitB + carry; // 求和
            carry = sum >= 2 ? 1 : 0; // 取进位
            sum = sum >= 2 ? sum - 2 : sum; // 若和大于2取余数；
            res = res.append(sum);结果拼接
            i--; // 从右往左遍历
            j--;
        }
        return res.reverse().toString(); // 因为从右边开始拼接，结果需反转
    }
}
```

## 代码实现（golang）

```go
func addBinary(a string, b string) string {
    res := ""
    i := len(a) - 1
    j := len(b) - 1
    carry := 0
    for i >= 0 || j >= 0 || carry != 0 {
        digitA := 0
        if i >= 0 {
            digitA = int(a[i] - '0')
        }
        digitB := 0
        if j >= 0 {
            digitB = int(b[j] - '0')
        }
        sum := digitA + digitB + carry
        carry = 0
        if sum >= 2 {
            carry = 1
        }
        sum = sum % 2
        res = strconv.Itoa(sum) + res
        i--
        j--
    }
    return res
}
```
