---
title: Minimum Size Subarray Sum
date: 2018-01-14 20:24:46
tags: 
  - algorithm
  - leetcode
---

### 问题描述

Given an array of **n** positive integers and a positive integer **s**, find the minimal length of a **contiguous** subarray of which the sum ≥ **s**. If there isn't one, return 0 instead.

For example, given the array `[2,3,1,2,4,3]` and `s = 7`,
the subarray `[4,3]` has the minimal length under the problem constraint.

More practice:

If you have figured out the *O*(*n*) solution, try coding another solution of which the time complexity is *O*(*n* log *n*).

<!-- more -->

### 分析

刚拿到这个问题，我的第一反应是用类似求最大子序列和的方式来进行相应的查找，但后来思考之后觉得并没有这个必要，我只要进行一次遍历就可以了，在遍历的过程中尾部会发生变化，我要做的只是把它的头记录下来，并在每次的循环中进行更新即可。

**记录这道题的主要目的倒不是因为它有多难，而是同为O(n)的做法，别人家的孩子写得比我要简洁得多，先记录下来作为提醒**

我的代码:

```C++
int minSubArray(int s, vector<int> &nums) {
  int start = 0, sum = 0, n = nums.size();
  int min_len = INT_MAX;
  
  for(int i = 0; i < n; i++) {
    sum += nums[i];
    if(sum < s)
      continue;
    
    for(int j = start; j < i; j++) {
      if(sum - nums[i] >= s) {
        sum -= nums[i];
        start = j + 1;
      }
    }
    
    min_len = min_len > i-start+1: i-start+1: min_len;
  }
  
  return min_len == INT_MAX? 0: min_len;
}
```

### 别人家的孩子的代码

```C++
int minSubArray(int s, vector<int> &nums) {
  int start = 0, sum = 0, n = nums.size(), min_len = INT_MAX;
  
  for(int i = 0; i < n; i++) {
    sum += nums[i];
    
    while(sum >= s) {
      min_len = min_len > i-start+1? i-start+1: min_len;
      
      sum -= nums[start++];
    }
  }
  return min_len == INT_MAX? 0: min_len;
}
```

*从上面可以看出虽然我们解法的基本思路是一致的，但别人家的孩子用循环来代替我迭代方法，简直赏心悦目*

#### 时间复杂度的简单分析

有些人可能看到for循环里面还包含着一个while循环，可能会觉得这个的时间复杂度并非是O(n)，但是我们要注意到在while里面的每次循环start只往前走了一步，也就是说**start的总前进步数只为n**，该循环的复杂度为O(n)，最终总的复杂度为O(n+n)=O(n)