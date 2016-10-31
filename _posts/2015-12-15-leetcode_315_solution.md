---
layout: post
comments: false
title: leetcode 315 解题报告
category: Algorithm
tag: algorithm leetcode
---

一道由逆序对引出的题，考查了对经典的归并排序算法的理解。

## 题目描述

### Count of Smaller Numbers After Self

You are given an integer array nums and you have to return a new counts array. The counts array has the property where counts[i] is the number of smaller elements to the right of nums[i].

Example:

```
Given nums = [5, 2, 6, 1]

To the right of 5 there are 2 smaller elements (2 and 1).
To the right of 2 there is only 1 smaller element (1).
To the right of 6 there is 1 smaller element (1).
To the right of 1 there is 0 smaller element.
```

Return the array [2, 1, 1, 0].

## 分析

题目要求每个数字右边比它小的数字的个数。初看有明显的暴力方法，每次扫描右边的数字，统计比它小的数字个数。时间复杂度是 $$O(n^2)$$。当然，这题出出来就不是用这个复杂度可以通过的，因此要优化。

想到一个数字与每个右边比它小的数字构成的都是一个逆序对，因此题目就转化成了如何求以每个数字开头的逆序对数。如果是求总的逆序对数，那么算法很简单了，用树状数组或归并排序统计都可以在 $$O(nlogn)$$ 的复杂度内完成。想到这里，应该可以想到是要把求总的逆序对数的算法加以改造，统计出以每个数开头的逆序对数。

我选择使用归并排序的方法。在归并排序合并的过程中，`i` 指向左边的部分，`j` 指向右边的部分。`i` 指向的数字记为 `[i]`， `j` 指向的数字记为 `[j]`。当 `[i]` 小于等于 `[j]` 时，没有逆序对产生，`i` 指针向后移。当 `[i]` 大于 `[j]` 时，在新的排好序的序列中，`[j]` 要出现在 `[i]` 之间，因此对于 `[i]` 以及它后面的所有数字，与 `[j]` 均构成了一个逆序对（因为根据归并排序的规则，左边的部分是排好序的，`[i]` 大于 `[j]` 那么 `i` 后面的数字一定都大于 `[j]`）。

数据结构方面，一个长度为 n 的数组 `result` 表示结果，`result[i]` 表示原序列中下标为 `i` 的数字后出现的比它小的数字个数。一个 struct 作为排序的基本单元，其中 `num` 表示原数字，`index` 表示该数字在原序列中的下标。在归并排序合并的过程中，加入一个计数器 `plus` ，表示进行到 `i` 时已经与 `[i]` 产生逆序对的数的数量。这样每次把 `[i]` 加入排序后的数组时，`result[[i].index]` 的值就加上 `plus`。完整的一次归并排序后就完成了统计。

总结一下算法：

1. 按输入序列顺序构造一个 struct 数组 `counts`，成员 `num` 为输入序列中数字的值，`index` 为原序列下标。
2. 构造一个数组 `result` 作为结果，下标和原序列下标一一对应，值为原序列中该数字后比它小的数字的个数。
3. 对 `counts` 归并排序，在合并的过程中进行以下操作：
    1. 令 `i` 指向左边的数组，`j` 指向右边的数组。计数器 `plus` 清零。
    2. 如果 `[i]` 小于等于 `[j]`，那么 `result[[i++].index] += plus`。
    3. 如果 `[i]` 大于 `[j]`，那么 `plus++`，`j++`。
    4. 重复上述过程直到左边或右边为空。
    5. 如果左边为空，则直接把右边剩余的部分全部加到结果序列中，本轮归并结束。
    6. 如果右边为空，`result[[i++].index] += plus` 直到左边为空，本轮归并结束。
4. 此时 `result` 数组即为所求。

## 心得

1. 进入归并排序的函数时，要判断左边界是否**小于**右边界，不满足直接退出。不满足的情况有两种，一是左边界等于右边界，此时说明已经分到单个元素了；二是左边界大于右边界，这种情况会在输入序列为空的时候出现，因为进入的时候左右边界分别是 0 和 `nums.size() - 1`，如果输入序列为空，则左右边界值分别为 0 和 -1。
2. 归并时的左中右划分。令 `mid = (l + r) / 2`，可不重不漏地把整个数组划分为 `[l, mid]` 和 `[mid + 1, r]` 两个部分。
3. 归并时需要构造一个与当前区间相同大小的数组，用来存放本轮合并后的序列。在退出之前要把这段序列复制回原序列，然后销毁。


