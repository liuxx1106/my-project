---
date: 2023-07-14 21:30:21
title: 剑指 Offer II 001. 整数除法
tags: [LeetCode, 整数除法]
categories: [剑指 Offer II]
index_img: https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/algorithm1.jpg    # 封面图
# banner_img: /img/post_banner.jpg  # 文章顶部大图
---
## 题目要求

[leetcode地址](https://leetcode.cn/problems/xoh6Oh/?envType=study-plan-v2&id=coding-interviews-special)

* 剑指 Offer II 001. 整数除法
* 简单
* 253
* 给定两个整数 a 和 b ，求它们的除法的商 a/b ，要求不得使用乘号 '*'、除号 '/' 以及求余符号 '%' 。

## 注意

* 整数除法的结果应当截去（truncate）其小数部分，例如：truncate(8.345) = 8 以及 truncate(-2.7335) = -2
* 假设我们的环境只能存储 32 位有符号整数，其数值范围是 [−2 ^ 31，2 ^ 31 -1 ]。本题中，如果除法结果溢出，则返回 2^31 − 1

 > 示例 1：
  输入：a = 15, b = 2
  输出：7
  解释：15/2 = truncate(7.5) = 7
  示例 2：
  输入：a = 7, b = -3
  输出：-2
  解释：7/-3 = truncate(-2.33333..) = -2
  示例 3：
  输入：a = 0, b = 1
  输出：0
  示例 4：
  输入：a = 1, b = 1
  输出：1

* 提示:
* -2 ^ 31 <= a, b <= 2 ^ 31 - 1
* b != 0
 */

## 思路

题目要求只能使用加减法，那我们自然想到用减法实现除法，用“被减数”能减去几次“减数”来衡量最后的结果，这时候我们想到求x的幂次的快速解法，将x成倍成倍的求幂，这里将减数成倍成倍的增大，次数对应也是成倍成倍的增大，例如：取a=23，b=2，b的变化如下:2->4->8->16,次数count的变化如下1->2->4->8,最后a-b=23-16=7，对7再执行一次上述过程，b:2->4,count:1->2,a-b=3, 然后对3再执行一次，b:2,count:1,a-b=1，1已经小于原b=2，可以结束了，最后计数一下每轮的count是多少8+2+1=11。

## 注意事项

为方便运算，我们需要将a，b都转为同正or同负，由于INT_MIN转正就越界了，我们只好都转负，这也是都转负的原因，有一种特殊情况 INT_MIN/(-1)就overflow了 所以直接特殊处理最终结果的正负。

## 代码实现（java）

```java
public static int divide(int a, int b) {
        //特殊情况直接处理
        if (a == Integer.MIN_VALUE && b == -1) {
            return Integer.MAX_VALUE;
        }
        if (b == 1) {
            return a;
        }
        int flag = 2; // 记录a,b取反标志，如只有一个取反，则最终结果也要取反
        if (a > 0) {
            flag--;
            a = -a;
        }
        if (b > 0) {
            flag--;
            b = -b;
        }
        int res = calc(a, b);
        return flag == 1 ? -res : res;

    }

    // 求a能减去b的次数
    private static int calc(int a, int b) {
        int res = 0;
        while (a <= b) { // a,b为负数
            int temp = b; // 临时变量
            int count = 1; // 记录a被减的次数，因为a>b,所以a一定可以被b减一次
            while (temp >= Integer.MIN_VALUE >> 1 && a <= temp << 1) { // temp应该大于等于最小值的二分之一，否则会导致溢出。a的绝对值应该比temp*2还小
                count += count; // 可以减的次数翻倍
                temp += temp; // 减数也翻倍
            }
            res += count; // 可以减的次数累加即为结果
            a -= temp; // a减去当前temp，重新去和b求最大可以减的次数
        }
        return res;
    }
```

## 代码实现（golang）

```go
func divide(a int, b int) int {
    //特殊情况直接处理
    if a == math.MinInt32 && b == -1 {
        return math.MaxInt32
    }
    if b == 1 {
        return a
    }
    flag := 2 // 记录a,b取反标志，如只有一个取反，则最终结果也要取反
    if a > 0 {
        flag--
        a = -a
    }
    if b > 0 {
        flag--
        b = -b
    }
    res := calc(a, b)
    if flag == 1 {
        return -res
    }
    return res
}

// 求a能减去b的次数
func calc(a int, b int) int {
    res := 0
    for a <= b { // a,b为负数
        temp := b // 临时变量
        count := 1 // 记录a被减的次数，因为a>b,所以a一定可以被b减一次
        for temp >= math.MinInt32/2 && a <= temp<<1 { // temp应该大于等于最小值的二分之一，否则会导致溢出。a的绝对值应该比temp*2还小
            count += count // 可以减的次数翻倍
            temp += temp // 减数也翻倍
        }
        res += count // 可以减的次数累加即为结果
        a -= temp // a减去当前temp，重新去和b求最大可以减的次数
    }
    return res
}
```
