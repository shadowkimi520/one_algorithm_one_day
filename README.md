# one_algorithm_one_day
每天一道算法题


> 为了应付面试，开始写博客，准备开三个系列，第一个 `JS算法系列`，主要整理一些面试过程中常见的算法题 ； 第二个 `ECMAScript规范解析`，主要是想从ES规范的角度来解析JS代码的实际执行过程，只有真正懂得了底层干了什么，才能完全理解JS，而不需要记住很多零散的坑； 第三个 `JS开源库代码阅读`，通过阅读开源代码加深对JS的理解，同事积累大型项目开发的最佳实践。


> 本系列为 `JS算法系列`

### _#_ 1. 求两个已排序数组的中位数

There are two sorted arrays nums1 and nums2 of size m and n respectively.

Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)). 

[具体参考leetCode](https://leetcode.com/problems/median-of-two-sorted-arrays/description/)
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

#### 分治法 Divide and Conquer

- 思路
    1. 假设两个数组共有len个元素，如果len为奇数，中位数为查找最终合并数组中的第 len/2+1 个元素；如果len为偶数，中位数为查找最终合并数组中的第 len/2 个 和 len/2+1 个元素。可以归结为TOP(k)问题, 如果解决了TOP(k)问题，那么求中位数不过是k取不同值的问题;
    2. 那么如何求TOP(k)问题呢？可以有以下两种方法

    方法一：取第一个数组的第p个元素，第二个数组的第q（保证 p+q===k，如果p确定了，q就随之确定了）个元素；如果第一个数组的第 p 个元素小于等于第二个数组的第 q 个元素，虽然我们不能确定第二个数组的第q个元素是不是我们要找的k元素，但是第一个数组的前p个元素一定不包含k元素（反证法：如果k元素为第一个数组第x1个元素，已知x1<=p，那么 x1+(q-1) <= p+q-1= k-1，此时x1对应的元素在最终合并的数组中最多为第k-1个元素，与我们的假设x1元素是K元素矛盾），因此可以抛弃第一个数组的前p个元素，形成一个较短的新数组；然后用新数组替换原先的第一个数组，再找这两个数组中的第k-p(更新k到k-p)个元素，依次递归。同理，如果第一个数组的第p个元素大于第二个数组的第q个元素，那么可以抛弃第二个数组的前q个元素。递归的终止条件包括两种情况：
    * 如果其中一个数组中的所有元素都被抛弃，直接返回另外一个数组的第k个元素；
    * 如果 k == 1， 直接返回第一个数组第一个元素和第二个数组第一个元素中的较小者；
    
    为了提高查找效率，采用折半查找，p设定为第 parseInt(k / 2) 个元素，这样每次淘汰的时候能淘汰掉（大约）一半的元素，从而使得递归的时候TOP(k)中的k的规模减半；对于中值问题，初始的时候k的大小为(m+n)/2，随后递归的时候每次K都减半，因此该算法的时候复杂度为O(log(m+n))

    ```js
    function findMedian(arr1, arr2) {
        var len1 = arr1.length, len2 = arr2.length, sum_len = len1 + len2;
        if (sum_len & 1) {
            return findKth(arr1, 0, arr2, 0, parseInt(sum_len / 2 + 1));
        }
        return (findKth(arr1, 0, arr2, 0, parseInt(sum_len / 2)) + findKth(arr1, 0, arr2, 0, parseInt(sum_len / 2 + 1))) / 2;
    }
    // 查找两个排序数组中的第K个元素
    function findKth(arr1, start1, arr2, start2, k) {
        if (start1 >= arr1.length) {
            return arr2[start2 + k - 1];
        }
        if (start2 >= arr2.length) {
            return arr1[start1 + k - 1];
        }
        if (k === 1) {
            return Math.min(arr1[start1], arr2[start2]);
        }
        var pos1 = start1 + parseInt(k / 2) - 1;
        var key1 = (pos1 >= arr1.length ? Number.POSITIVE_INFINITY ：arr1[pos1]);

        var pos2 = start2 + parseInt(k / 2) - 1;
        var key2 = (pos2 >= arr2.length ? Number.POSITIVE_INFINITY : arr2[pos2]);

        if (key1 <= keys2) {
            return findKth(arr1, pos1 + 1, arr2, start2, k - parseInt(k / 2));
        } else {
            return findKth(arr1, start1, arr2, pos2 + 1, k - parseInt(k / 2));
        }
        
    }
    ```
    > NOTE1: JS中数字均为浮点数，因此需要使用 parseInt 取整
    > NOTE2: findKth函数的代码中第一个数组的第p个元素的下标是pos1, 它是该数组中的第 pos1-start1+1 个元素，即第parseInt(k / 2)个元素；同理，第二个数组中的第q个元素为第二个数组中的第 parseInt(k / 2)个元素，此时p+q = 2 * parseInt(k / 2)， 当k为基数时 2 * parseInt(k / 2) 比 k 小1， 而我们的假设中p + q === k，之所以上述代码可以得到正确的结果是因为 k - 1 <= p + q <= k，只要p+q 不大于k，被淘汰掉的parseInt(k / 2)个元素中就不可能包含TOP(K)元素，只不过是有时候在一次递归过程中少淘汰一个元素而已。下面的代码我们严格按照 p + q === k来执行淘汰，也能得到正确的结果（仅修改了添加注释的两行代码）：
    ```js
    var pos1 = start1 + parseInt(k / 2) - 1;
    var key1 = (pos1 >= arr1.length ? Number.POSITIVE_INFINITY ：arr1[pos1]);

    var pos2 = start2 + k - parseInt(k / 2) - 1;   // 使得 p + q 严格等于 k
    var key2 = (pos2 >= arr2.length ? Number.POSITIVE_INFINITY : arr2[pos2]);

    if (key1 <= keys2) {
        return findKth(arr1, pos1 + 1, arr2, start2, k - parseInt(k / 2));
    } else {
        return findKth(arr1, start1, arr2, pos2 + 1, parseInt(k / 2)); // 淘汰掉的元素个数要相应变更
    }
    ```

    方法二： 分别找出两个数组的中位数进行比较

    1. 如果arr1[mid1] <= arr2[mid2]，此时又分两种情况：如果从两个数组开始到中位数（包括中位数）的元素个数之和大于k，就表示第K大的元素一定不在arr2[mid2...end2]当中，因此可以将它们淘汰掉；如果从两个数组开始到中位数（包括中位数）的元素个数之和小于等于k，就表示arr1[beg1...mid1]元素一定属于第k大元素之前的部分，即这一部分一定比第k大元素来的小，因此可以将它们先排除掉，问题就缩小为从剩下的数组中查找TOP(k-(mid1-beg1+1))，可以递归求解；
    2. 如果arr1[mid1] > arr2[mid2], 此时又可以分为两种情况：如果从两个数组开始到中位数（包括中位数）的元素个数之和大于k，就表示第K大的元素一定不在arr1[mid1...end1]当中，因此可以将它们淘汰掉；如果从两个数组开始到中位数（包括中位数）的元素个数之和小于等于k，就表示arr2[beg2...mid2]元素一定属于第k大元素之前的部分，即这一部分一定比第k大元素来的小，因此可以将它们先排除掉，问题就缩小为从剩下的数组中查找TOP(k-(mid1-beg1+1))，可以递归求解；
    
    代码如下：
    ```js
    function findMedian(arr1, arr2) {
        var len1 = arr1.length, len2 = arr2.length, sum_len = len1 + len2;
        if (sum_len & 1) {
            return findKth(arr1, 0, len1 - 1, arr2, 0, len2 - 1, parseInt(sum_len / 2 + 1));
        }
        return (findKth(arr1, 0, len1 - 1, arr2, 0, len2 - 1, parseInt(sum_len / 2)) + findKth(arr1, 0, len1 - 1, arr2, 0, len2 - 1, parseInt(sum_len / 2 + 1))) / 2;
    }
    // 查找两个排序数组中的第K个元素
    function findKth(arr1, beg1, end1, arr2, beg2, end2, k) {
        if (beg1 > end1) {
            return arr2[beg2 + k - 1];
        }
        if (beg2 > end2) {
            return arr1[beg1 + k - 1];
        }
        var mid1 = parseInt((beg1 + end1) / 2), mid2 = parseInt((beg2 + end2) / 2);
        if( arr[mid1] <= arr2[mid2] ) {
            if( k < (mid1 - beg1) + (mid2 - beg2) + 2 ) {
                return findKth(arr1, beg1, end1, arr2, beg2, mid2 - 1, k );
            }
            else {
                return findKth(arr1, mid1 + 1, end1, arr2, beg2, end2, k- (mid1 - beg1) - 1);            
            }
        }
        else {
            if( k < (mid1 - beg1) + (mid2 - beg2) + 2 ) {
                return findKth(arr1, beg1, mid1 - 1, arr2, beg2, end2, k );
            }
            else {
                return findKth(arr1, beg1, end1, arr2, mid2 + 1, end2, k - (mid2 - beg2) - 1 );
            }
        }     
    }
    ```

- 总结

方法1主要是保证在递归的时候TOP(k)中k的规模减半，能保证p+q<=k; 方法2在两个数组中均进行折半查找，因此不能保证两个数组中位数之前的元素个数和小于等于k，即不能保证p+q <= k，只有在p+q<= k的时候才缩减TOP(k)的规模，所以递归结束的标志只能是其中一个数组的元素都被淘汰掉，其事件复杂度为log(m)+log(n)，即log(m*n)，下图是一个测试案例，每个数组均有16个元素，findKth查找第16个元素的时候需要执行9次，案例中打印出了每次起止位置的变化。

![](https://github.com/shadowkimi520/one_algorithm_one_day/raw/master/images/two_sorted_array_get_median.png) 

还有一种更精妙的方法，采用循环替代递归，具体参考 [【分步详解】两个有序数组中的中位数和Top K问题](http://blog.csdn.net/hk2291976/article/details/51107778)