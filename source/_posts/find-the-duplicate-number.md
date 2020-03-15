---
title: Find the Duplicate Number
date: 2018-01-14 14:58:47
tags: 
  - algorithm
  - leetcode
---

### 问题描述:

Given an array *nums* containing *n* + 1 integers where each integer is between 1 and *n* (inclusive), prove that at least one duplicate number must exist. Assume that there is only one duplicate number, find the duplicate one.

Note:

1. You **must not** modify the array (assume the array is read only).
2. You must use only constant, *O*(1) extra space.
3. Your runtime complexity should be less than `O(n2)`.
4. There is only one duplicate number in the array, but it could be repeated more than once.

<!-- more -->

### 解题思路

首先，最简单粗暴的方法就是直接排序之后遍历查找就行，复杂度大概在O(nlogn)这里，然而很明显这道题就是摆明了不让你用排序了（当然，经过本人验证，如果你直接用排序，它好像也没把你怎么样）。

这样想来，想要在$O(n^2)$之内的复杂度完成上述任务的话，必须得想办法去掉一些不必要的比较。重回该题我们会发现，其实这道题就是鸽巢原理的证明，对于1~n-1的数中有n个值，那么我们可以考虑取1~n-1中的中位数，并统计小于它的个数$n_1$和大于它的个数$n_2$。

1. $n_1\gt n_2$: 重复的值在左半区间
2. $n_2\gt n_1$：重复的值在右半区间

从而在某种意义上实现了二分查找，这样能够在O(1)的空间消耗下实现O(nlgn)的时间复杂度。

示例代码如下:

```C++
int findDuplicate(vector<int> &nums) {
  int size = nums.size();
  int low = 1, hight = size - 1;
  
  int mid = 0;
  while(low < high) {
    mid = (low + high) / 2;
    
    int count = 0;
    for(auto item: nums) {
      if(item <= mid) count++;
    }
    
    if(count > mid) {
      high = mid;
    } else {
      low = mid + 1;
    }
  }
  return low;
}
```

后记：当然了，当我提交过后发现了有一大波人在时间复杂度上吊打我之后，我就怀着崇敬的心情看了下别人家的孩子的代码（此处献上我的膝盖）。

（之后再补充别人家的孩子的代码）

### 别人家的孩子

观察此题目，我们可以看出，其下标为0~n-1，其取值也恰好为1~n-1，因此其实每个下标对应的值同时也索引到了另外一个元素，如果没有任何元素有重复的话，根据上述映射关系形成的应该类似于单链表，但一旦出现了重复的元素，就会导致索引的过程中出现了环，那么这道题的解决过程又变成了找环。

我们可以先定义两个“指针”——slow, fast，其中slow一次只跳跃一个step，fast一次跳跃两个step，当slow==fast时，我们可以认为这两个指针此时处在环之中。之后我们从头开始进行，同时一次更新一个step，那么当slow和fast再次相遇时，就是到了这个环的入口点，即重复的元素。

示例代码如下：

```C++
int findDuplicate(vector<int> &nums) {
  int slow = nums[0];
  int fast = nums[nums[0]];
  
  while(slow != fast) {
    slow = nums[slow];
    fast = nums[nums[fast]];
  }
  
  fast = 0;
  while(slow != fast) {
    slow = nums[slow];
    fast = nums[fast];
  }
  
  return slow;
}
```

