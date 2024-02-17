---
date: 2024-01-12 21:00:00
title: 剑指 Offer II 059. 数据流中的第 K 大元素
tags: [LeetCode, 堆]
categories: [剑指 Offer II]
index_img: https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/algorithm059.png   # 封面图
# banner_img: /img/post_banner.jpg  # 文章顶部大图
---

## 题目要求

[leetcode地址](https://leetcode.cn/problems/jBjn9C/description/?envType=study-plan-v2&envId=coding-interviews-special)

设计一个找到数据流中第 k 大元素的类（class）。注意是排序后的第 k 大元素，不是第 k 个不同的元素。

请实现 KthLargest 类：

- KthLargest(int k, int[] nums) 使用整数 k 和整数流 nums 初始化对象。
- int add(int val) 将 val 插入数据流 nums 后，返回当前数据流中第 k 大的元素。

#### 示例

输入：
["KthLargest", "add", "add", "add", "add", "add"]
[[3, [4, 5, 8, 2]], [3], [5], [10], [9], [4]]
输出：
[null, 4, 5, 5, 8, 8]

解释：
KthLargest kthLargest = new KthLargest(3, [4, 5, 8, 2]);
kthLargest.add(3);   // return 4
kthLargest.add(5);   // return 5
kthLargest.add(10);  // return 5
kthLargest.add(9);   // return 8
kthLargest.add(4);   // return 8

#### 提示

- 1 <= k <= 104
- 0 <= nums.length <= 104
- -104 <= nums[i] <= 104
- -104 <= val <= 104
- 最多调用 add 方法 104 次
- 题目数据保证，在查找第 k 大元素时，数组中至少有 k 个元素

## 思路及步骤

本题比较好理解，即每次插入元素，然后获取排序后的第K大元素。
我们可以利用小顶堆的性质，父节点元素比子节点每一个元素都小。因此我们初始化一个堆heap,容量为K,则根节点元素为第K大元素。每次插入新的元素，当heap.size()>k,则删除堆顶元素，维护这样的一个小顶堆，堆顶元素即为第K大元素。

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
class KthLargest {

    private PriorityQueue<Integer> heap;
    private int k;
    // 初始化，将所有的元素都插入堆中
    public KthLargest(int k, int[] nums) {
        this.k = k;
        heap = new PriorityQueue<>(k);
        for(int num :nums){
            heap.add(num);
        }
    }
    
    public int add(int val) {
        // 将元素插入堆中
        heap.add(val);
        // 如果堆的大小超过 k，则将堆顶的元素删除
        while (heap.size() > k){
            heap.poll();
        }
        // 最后，堆顶就是 TopK 元素 
        return heap.peek();
    }
}
```

## 代码实现（golang）

```go
type KthLargest struct {
    sort.IntSlice
    k int
}

func Constructor(k int, nums []int) KthLargest {
    kl := KthLargest{k: k}
    for _, val := range nums {
        kl.Add(val)
    }
    return kl
}

func (kl *KthLargest) Push(v interface{}) {
    kl.IntSlice = append(kl.IntSlice, v.(int))
}

func (kl *KthLargest) Pop() interface{} {
    a := kl.IntSlice
    v := a[len(a)-1]
    kl.IntSlice = a[:len(a)-1]
    return v
}

func (kl *KthLargest) Add(val int) int {
    heap.Push(kl, val)
    if kl.Len() > kl.k {
        heap.Pop(kl)
    }
    return kl.IntSlice[0]
}
```
