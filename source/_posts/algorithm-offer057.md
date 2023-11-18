---
title: 剑指 Offer II 057. 存在重复元素 III
tags: [LeetCode, 二叉搜索树]
categories: [剑指 Offer II]
index_img: https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/algorithm057.png   # 封面图
# banner_img: /img/post_banner.jpg  # 文章顶部大图
---

## 题目要求

[leetcode地址](https://leetcode.cn/problems/7WqeDu/?envType=study-plan-v2&envId=coding-interviews-special)
给你一个整数数组 nums 和两个整数 k 和 t 。请你判断是否存在 两个不同下标 i 和 j，使得 abs(nums[i] - nums[j]) <= t ，同时又满足 abs(i - j) <= k 。

如果存在则返回 true，不存在返回 false。

#### 示例1

> 输入: nums = [1,2,3,1], k = 3, t = 0
> 输出: true

#### 示例2

> 输入: nums = [1,0,1,1], k = 1, t = 2
> 输出: true

#### 示例3

> 输入: nums = [1,5,9,1,5,9], k = 2, t = 3
> 输出: false

#### 提示

- 0 <= nums.length <= 2 * 10^4
- 2^31 <= nums[i] <= 2^31 - 1
- 0 <= k <= 10^4
- 0 <= t <= 2^31 - 1

## 思路及步骤

步骤如下：

- 首先，定义一个有序的 TreeSet 集合 set，用来存储元素。
- 然后，遍历整数数组 nums。
- 在每次遍历过程中，首先找到 set 中大于等于 (nums[i] - t) 的最小元素，使用 ceiling 变量进行保存。
- 如果 ceiling 不为空，并且它的值小于等于 (nums[i] + t)，则说明找到了满足条件的两个元素，返回 true。
- 否则，将当前元素 (long) nums[i] 添加到 set 中。
- 如果当前索引 i 大于等于 k，即窗口大小已经达到了 k+1，则需要移除窗口最左边的元素 (long) nums[i - k]，以保持窗口大小不超过 k。
- 最后，如果遍历完整个数组都未找到符合条件的元素对，则返回 false。

这段代码利用了 TreeSet 的排序和查找特性，通过维护一个窗口大小为 k 的集合，在遍历过程中判断是否存在满足条件的元素对。时间复杂度为 O(nlogk)，其中 n 是数组的长度。

## 代码实现（java）

```java
public boolean containsNearbyAlmostDuplicate(int[] nums, int k, int t) {
    int n = nums.length;
    TreeSet<Long> set = new TreeSet<Long>(); // 创建一个有序的 TreeSet 集合来存储元素
    for (int i = 0; i < n; i++) { // 遍历数组
        Long ceiling = set.ceiling((long) nums[i] - (long) t); // 找到集合中大于等于 (nums[i] - t) 的最小元素
        if (ceiling != null && ceiling <= (long) nums[i] + (long) t) { // 如果该元素存在且小于等于 (nums[i] + t)
            return true; // 则找到了符合条件的元素对，返回 true
        }
        set.add((long) nums[i]); // 将当前元素添加到集合中
        if (i >= k) { // 如果 i 大于等于 k，即窗口大小已经达到了 k+1
            set.remove((long) nums[i - k]); // 移除窗口最左边的元素，保持窗口大小不超过 k
        }
    }
    return false; // 遍历完数组都未找到符合条件的元素对，返回 false
}
```

## 代码实现（golang）

```go
func containsNearbyAlmostDuplicate(nums []int, k int, t int) bool {
 n := len(nums)
 buckets := make(map[int64]int64) // 创建一个 map 来存储桶和元素的映射关系

 for i := 0; i < n; i++ {
  num := int64(nums[i]) 
  bucketNum := getBucketNumber(num, int64(t)+1) // 计算当前元素所属的桶号

  if _, exists := buckets[bucketNum]; exists { // 如果当前桶已经存在元素，则找到了满足条件的元素对
   return true
  }

  if val, exists := buckets[bucketNum-1]; exists && num-val <= int64(t) { // 检查前一个桶中是否存在满足条件的元素
   return true
  }

  if val, exists := buckets[bucketNum+1]; exists && val-num <= int64(t) { // 检查后一个桶中是否存在满足条件的元素
   return true
  }

  buckets[bucketNum] = num // 将当前元素添加到对应的桶中

  if i >= k { // 如果超过索引范围 k，需要删除最早添加的元素所属的桶
   delete(buckets, getBucketNumber(int64(nums[i-k]), int64(t)+1))
  }
 }

 return false // 遍历完整个数组都未找到符合条件的元素对，则返回 false
}

func getBucketNumber(num, bucketSize int64) int64 {
 if num >= 0 {
  return num / bucketSize // 对于非负数，直接计算桶号
 }
 return (num+1)/bucketSize - 1 // 对于负数，需要将计算结果减一
}
```

> 这个优化的实现使用了桶排序的思想，将数据根据具有固定范围的桶进行分组。每个桶的范围是 t+1，其中 t 是差值范围。

在遍历数组时，对于每个元素，我们计算它所属的桶号，并检查当前桶和相邻的桶中是否存在满足条件的元素。如果存在，即找到了满足条件的两个元素，返回 true。

然后，将当前元素添加到对应的桶中，并在超过索引范围 k 后删除最早添加的元素所属的桶。

相比之前的实现，这个优化的版本避免了使用 TreeSet 的模拟和循环遍历差值范围内的所有值，而是根据桶的范围来判断是否存在符合条件的元素对。这样可以提高算法的效率。

该优化版本的时间复杂度为 O(n)，其中 n 是数组的长度，与差值范围 t 无关。
