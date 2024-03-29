---
title: 在一维数组算法题中使用双指针
date: 2021-10-20T14:15:41+08:00
typora-root-url: ../../static
categories:
  - tech
series:
  - Algorithm
tags:
  - algorithm
  - array
  - list
  - pointer
---

## 什么是双指针

双指针就是指为数组定义两个指针，通过移动两个指针来读取数组相应位置的数据，并对数据进行处理。这种方式也可以根据题型不同推广到「三指针」甚至「多指针」，但是因为理念上是相同的，多指针的题型徒增题目的复杂度，所以通常面试类问题中出现「双指针」的概率更大。

## 题型特征

那么如何判断一道题目适合使用双指针作为解题方式呢？我们先举几个例子[^examples-from]：

- [Remove Element - LeetCode](https://leetcode.com/problems/remove-element/)
- [Remove Duplicates from Sorted Array - LeetCode](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)
- [Move Zeroes - LeetCode](https://leetcode.com/problems/move-zeroes/)
- [Squares of a Sorted Array - LeetCode](https://leetcode.com/problems/squares-of-a-sorted-array/)
- [Minimum Size Subarray Sum - LeetCode](https://leetcode.com/problems/minimum-size-subarray-sum/)

解决这类题目的直观想法是题目可以直接通过定义两个下标 `i` 和 `j`，移动下标实现内外两个迭代。在这样的解法中，通常在两个迭代过程中 `i` 是不会走回头路的，但是 `j` 一直会回到 `i` 当前或之后的位置。所以这类题型会有另一个特征就是**不走回头路**，下标 `j` 在 `i` 移动时，应该保持当前位置或者继续向右前进。

在单纯的两层循环中，复杂度只能控制在 $O(NlogN)$。而使用双指针的题型中，因为两个指针都只从左到右移动一轮，所以复杂度只有 $O(2N)$，即 $O(N)$。

如果一道数组类算法题只需要两层循环就可以解决，那它就没有的意义了。所以遇到一道数组类型的题目，可以先将问题简化成两层循环，再基于这样的解决将思路转换为两个指针，并思考基于双指针，是否能进一步优化解答。

[^examples-from]: 例子参考了[代码随想录](https://programmercarl.com/0027.%E7%A7%BB%E9%99%A4%E5%85%83%E7%B4%A0.html)。

