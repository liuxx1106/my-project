---
date: 2024-02-02 22:30:21
title: LCR 061. 查找和最小的 K 对数字
tags: [LeetCode, 堆]
categories: [剑指 Offer II]
index_img: https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/algorithm061.png   # 封面图
# banner_img: /img/post_banner.jpg  # 文章顶部大图
---

## 题目要求

[leetcode地址](https://leetcode.cn/problems/qn8gGX/description/?envType=study-plan-v2&envId=coding-interviews-special)

给定两个以升序排列的整数数组 nums1 和 nums2 , 以及一个整数 k 。

定义一对值 (u,v)，其中第一个元素来自 nums1，第二个元素来自 nums2 。

请找到和最小的 k 个数对 (u1,v1),  (u2,v2)  ...  (uk,vk) 。

#### 示例
> 示例 1:

输入: nums1 = [1,7,11], nums2 = [2,4,6], k = 3
输出: [1,2],[1,4],[1,6]
解释: 返回序列中的前 3 对数：
    [1,2],[1,4],[1,6],[7,2],[7,4],[11,2],[7,6],[11,4],[11,6]
> 示例 2:

输入: nums1 = [1,1,2], nums2 = [1,2,3], k = 2
输出: [1,1],[1,1]
解释: 返回序列中的前 2 对数：
     [1,1],[1,1],[1,2],[2,1],[1,2],[2,2],[1,3],[1,3],[2,3]
> 示例 3:

输入: nums1 = [1,2], nums2 = [3], k = 3 
输出: [1,3],[2,3]
解释: 也可能序列中所有的数对都被返回:[1,3],[2,3]

#### 提示

- 1 <= nums1.length, nums2.length <= 10^4
- -10^9 <= nums1[i], nums2[i] <= 10^9
- nums1, nums2 均为升序排列
- 1 <= k <= 1000


#### 思路1

大顶堆

由于 nums1 和 nums2 都是升序排序的，所以最小的k个数对肯定是在 nums1[0,k-1] 和 nums2[0,k-1] 中配对的。

1. 创建一个大根堆，堆中元素排序按照数对和进行逆序排序。
2. nums1 取前k个数（长度 n 小于 k 则取 n 个，nums2 相同）与数组2取前 k 个数组成 k*k 个数对。
3. 每次都将当前配对的数对插入堆中，然后判断堆中元素总数是否超过 k , 如果超过 k 则将堆顶元素删除。
4. 不断重复 3 操作，最后剩下的堆中的数对，就是和最小的 k 个数对。
时间复杂度：O(k^2logk) ，其中容量为k的堆的添加与删除是O(logK),循环k^2次，为O(k^2logk)。

#### 思路2

小顶堆

1. 创建一个小顶堆heap，存放[nums1中数字的索引，nums中数字的索引]；
2. 将nums1中的数字的索引预先存入heap,即heap.add(i,0)，0 < i < Math.min(k,nums1.length);
3. 当k>0且堆中元素不为空时，每次取出堆顶元素{i，j}，放入结果ans中;
4. 当pair[1] + 1 < nums2.length,将[i,j+1]加入堆中
5. 一直重复 3-4 过程，直至选到 k 个数对。

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

## 代码实现

### 思路一

```java
class Solution {
    public List<List<Integer>> kSmallestPairs(int[] nums1, int[] nums2, int k){
        // 创建一个大根堆，堆中元素排序按照数对和进行逆序排序。
        PriorityQueue<List<Integer>> heap = new PriorityQueue<>(
                (pair1, pair2) -> pair2.get(0) + pair2.get(1) - pair1.get(0) - pair1.get(1)
        );
        // 数组1取前k个数 （长度n小于k则取n个，数组2也相同）,组成 k*k 个数对
        int len1 = Math.min(nums1.length, k);
        int len2 = Math.min(nums2.length, k);
        for(int i = 0; i < len1; i++){
            for(int j = 0; j < len2; j++){
                // 将数对加入大根堆中
                ArrayList<Integer> list = new ArrayList<>();
                list.add(nums1[i]);
                list.add(nums2[j]);
                heap.add(list);
                // 如果大根堆中的元素总量超过 k , 则将大根堆的堆顶元素删除。
                if(heap.size() > k) heap.poll();
            }
        }
        // 最后剩下的大根堆中的数对就是和最小的k个数对，将其保存到列表中。
        ArrayList<List<Integer>> ans = new ArrayList<>();
        Iterator<List<Integer>> iterator = heap.iterator();
        while (iterator.hasNext()){
            ans.add(iterator.next());
        }
        return ans;
    }
}
```

### 思路二

```java
class Solution {
    public List<List<Integer>> kSmallestPairs(int[] nums1, int[] nums2, int k) {
        // 创建一个小根堆，小根堆中存放 <nums1对应元素的索引，nums2对应元素的索引>
        ArrayList<List<Integer>> ans = new ArrayList<>();
        PriorityQueue<int[]> heap = new PriorityQueue<>(
                (pair1, pair2) -> nums1[pair1[0]] + nums2[pair1[1]] - nums1[pair2[0]] - nums2[pair2[1]]
        );
        // 对小根堆进行初始化，将 <0,0>, ... , <k-1,0> 插入栈中（如果长度 n 小于 k 则取 n 个）。
        for(int i = 0; i < Math.min(k, nums1.length); i++){
            heap.add(new int[]{i,0});
        }

        for ( ; k > 0 && !heap.isEmpty(); k--){
            // 选出和最小的数对【i,j】(堆顶)，将堆顶弹出，把 `{nums1[i], nums2[j]}` 保存到列表中。
            int[] pair = heap.poll();
            ans.add(Arrays.asList(nums1[pair[0]],nums2[pair[1]]));
            // 当 `j + 1 < nums2.length` 时， 才将【i,j+1】插入堆中
            if(pair[1] < nums2.length - 1){
                heap.add(new int[]{pair[0], pair[1] + 1});
            }
            
        }
        // 返回结果
        return ans;
    }
}
```