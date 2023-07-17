---
title: 剑指 Offer II 006. 排序数组中两个数字之和
tags: [LeetCode, 二分查找， 双指针]
categories: [剑指 Offer II]
index_img: /img/index_img/algorithm4.jpg    # 封面图
# banner_img: /img/post_banner.jpg  # 文章顶部大图
---

## 题目要求

[leetcode地址](https://leetcode.cn/problems/kLl5u1/?envType=study-plan-v2&envId=coding-interviews-special)
给定一个已按照 升序排列  的整数数组 numbers ，请你从数组中找出两个数满足相加之和等于目标数 target 。

函数应该以长度为 2 的整数数组的形式返回这两个数的下标值。numbers 的下标 从 0 开始计数 ，所以答案数组应当满足 0 <= answer[0] < answer[1] < numbers.length 。

假设数组中存在且只存在一对符合条件的数字，同时一个数字不能使用两次。

#### 示例1
>
> 输入：numbers = [1,2,4,6,10], target = 8
输出：[1,3]
解释：2 与 6 之和等于目标数 8 。因此 index1 = 1, index2 = 3 。

#### 示例2
>
> 输入：numbers = [2,3,4], target = 6
输出：[0,2]

#### 示例3
>
> 输入：numbers = [-1,0], target = -1
输出：[0,1]

#### 提示

2 <= numbers.length <= 3 * 104
-1000 <= numbers[i] <= 1000
numbers 按 非递减顺序 排列
-1000 <= target <= 1000
仅存在一个有效答案

## 思路

- 1. 由于给定数组为有序排列，所以第一想到的应该是二分查找，遍历数组，然后通过二分查找去找到target-numbers[i]。时间复杂度O(nlogn),空间复杂度O(1)。
- 2. 由于给定数组为有序排列，所以可以用双指针从数组两边扫描不断逼近正确答案。时间复杂度为O(n), 空间复杂度为O(1);

## 代码实现（java）

```java
class Solution {
 //方法1. 二分查找
    // public int[] twoSum(int[] numbers, int target) {
    //         if(numbers.length == 0 || numbers == null) return new int[0];
    //         for(int i = 0; i<numbers.length;i++) {
    //             int t = target - numbers[i];
    //             int index = binarySearch(numbers, t, i+1, numbers.length-1);
    //             if(index != -1) {
    //                 return new int[]{i, index};
    //             }

    //         }
    //         return new int[0];
    // }
    // int binarySearch(int[] numbers, int t,int left,int right) {
    //     while(left <= right) {
    //         int mid = (left + right) >> 1;
    //         if(numbers[mid] == t) {
    //             return mid;
    //         }else if(numbers[mid] > t) {
    //             right = mid -1;
    //         }else if(numbers[mid] < t) {
    //             left = mid +1;
    //         }
    //     }
    //     return -1;
    // }

 //方法2. 双指针
    // public int[] twoSum(int[] numbers, int target) {
    //     if(numbers.length == 0 || numbers == null) return new int[0];
    //         
    //         int left = 0;
    //         int right = numbers.length -1;
    //         while(left<right) {
    //             int sum = numbers[left] + numbers[right];
    //             if(sum == target) {
    //                 return new int[]{left,right};
    //             } else if(sum<target) {
    //                 left++;
    //             } else{
    //                 right--;
    //             }
    //         }
    //         return new int[0];
    // }
    // 方法3. hashmap
    public int[] twoSum(int[] numbers, int target) {
        Map<Integer, Integer> maps = new HashMap();
        for(int i= 0;i<numbers.length;i++) {
            if(maps.containsKey(target-numbers[i])) {
                return new int[]{maps.get(target-numbers[i]),i};
            }
            maps.put(numbers[i],i);
        }
        return new int[0];
    }
}
```

## 代码实现（golang）

```go
//1.map解法
func twoSum(numbers []int, target int) []int {
    maps := make(map[int]int)
    for i, num := range numbers {
        if j, ok := maps[target-num]; ok {
            return []int{j, i}
        }
        maps[num] = i
    }
    return []int{}
}

// 2. 二分查找
func twoSum(numbers []int, target int) []int {
    if len(numbers) == 0 {
        return []int{}
    }
    for i := 0; i < len(numbers); i++ {
        t := target - numbers[i]
        index := binarySearch(numbers, t, i+1, len(numbers)-1)
        if index != -1 {
            return []int{i, index}
        }
    }
    return []int{}
}

func binarySearch(numbers []int, t, left, right int) int {
    for left <= right {
        mid := (left + right) >> 1
        if numbers[mid] == t {
            return mid
        } else if numbers[mid] > t {
            right = mid - 1
        } else if numbers[mid] < t {
            left = mid + 1
        }
    }
    return -1
}

//3.双指针
func twoSum(numbers []int, target int) []int {
    if len(numbers) == 0 {
        return []int{}
    }

    left, right := 0, len(numbers)-1

    for left < right {
        sum := numbers[left] + numbers[right]
        if sum == target {
            return []int{left, right}
        } else if sum < target {
            left++
        } else {
            right--
        }
    }

    return []int{}
}
```
