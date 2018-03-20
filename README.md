# one_algorithm_one_day
每天一道算法题


> 为了应付面试，开始写博客，准备开三个系列，第一个 `JS算法系列`，主要整理一些面试过程中常见的算法题 ； 第二个 `ECMAScript规范解析`，主要是想通过从ES规范的角度来解析JS代码的实际执行过程，只有真正懂得了底层干了什么，才能完全理解JS，而不需要记住很多零散的坑； 第三个 `JS开源库代码阅读`，通过阅读开源代码加深对JS的理解，同事积累大型项目开发的最佳实践。


> 本系列为 `JS算法系列`

### _#_ 1. 求两个已排序数组的中位数

There are two sorted arrays nums1 and nums2 of size m and n respectively.

Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).
```js
Example 1:
nums1 = [1, 3]
nums2 = [2]

The median is 2.0

Example 2:
nums1 = [1, 2]
nums2 = [3, 4]

The median is (2 + 3)/2 = 2.5
```