---
date: 2024-01-12 21:00:00
title: LCR 060. 前 K 个高频元素
tags: [LeetCode, 堆]
categories: [剑指 Offer II]
index_img: https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/algorithm059.png   # 封面图
# banner_img: /img/post_banner.jpg  # 文章顶部大图
---

## 题目要求

[leetcode地址](https://leetcode.cn/problems/g5c51o/description/?envType=study-plan-v2&envId=coding-interviews-special)

给定一个整数数组 nums 和一个整数 k ，请返回其中出现频率前 k 高的元素。可以按 任意顺序 返回答案。

#### 示例

输入: nums = [1,1,1,2,2,3], k = 2
输出: [1,2]
示例 2:

输入: nums = [1], k = 1
输出: [1]

#### 提示

- 1 <= nums.length <= 105
- k 的取值范围是 [1, 数组中不相同的元素的个数]
- 题目数据保证答案唯一，换句话说，数组中前 k 个高频元素的集合是唯一的

#### 进阶

所设计算法的时间复杂度 必须 优于 O(n log n) ，其中 n 是数组大小。

#### 思路

哈希表+小顶堆

1. 首先，将数组中的数字及出现次数存入hash表，有Map<num,num出现的次数>；
2. 以num出现的次数为比较对象，维护一个小顶堆heap，遍历map将其中的元素以数组的形式`[num,num出现的次数]`不断添加到小顶堆，
   当heap.size()>K时，弹出最顶端元素（只返回前K大元素，而小顶堆顶端元素最小）；
3. 从heap中取出元素num添加到数组返回。

## java中堆的实现

```java
//小根堆，默认容量11
PriorityQueue<Integer> minHeap = new PriorityQueue<Integer>();
//大根堆，容量11
PriorityQueue<Integer> maxHeap = new PriorityQueue<Integer>(11,new Comparator<Integer>(){
    @Override
    public int compare(Integer i1,Integer i2){
        return i2-i1;
    }
});
```

## 代码实现（java）

```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        // 哈希表存储
        HashMap<Integer, Integer> map = new HashMap<>();
        for (int num : nums) {
            map.put(num, map.getOrDefault(num, 0) + 1);
        }
        // 创建小顶堆
        PriorityQueue<int[]> heap = new PriorityQueue<int[]>(new Comparator<int[]>() {
            public int compare(int[] m, int[] n) {
                return m[1] - n[1];
            }
        });

        for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
            heap.add(new int[] { entry.getKey(), entry.getValue() });
            if (heap.size() > k) {
                heap.poll();
            }
        }
        int[] res = new int[k];
        for (int i = 0; i < k; i++) {
            res[i] = heap.poll()[0];
        }
        return res;
    }
}
```

## 代码实现（golang）

```go
func topKFrequent(nums []int, k int) []int {
    occurrences := map[int]int{}
    for _, num := range nums {
        occurrences[num]++
    }
    h := &IHeap{}
    heap.Init(h)
    for key, value := range occurrences {
        heap.Push(h, [2]int{key, value})
        if h.Len() > k {
            heap.Pop(h)
        }
    }
    ret := make([]int, k)
    for i := 0; i < k; i++ {
        ret[k - i - 1] = heap.Pop(h).([2]int)[0]
    }
    return ret
}

type IHeap [][2]int

func (h IHeap) Len() int           { return len(h) }
func (h IHeap) Less(i, j int) bool { return h[i][1] < h[j][1] }
func (h IHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

func (h *IHeap) Push(x interface{}) {
    *h = append(*h, x.([2]int))
}

func (h *IHeap) Pop() interface{} {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[0 : n-1]
    return x
}
```
