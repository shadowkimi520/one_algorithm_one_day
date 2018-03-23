# one_algorithm_one_day
每天一道算法题

> 为了应付面试，开始写博客，准备开三个系列，第一个 `JS算法系列`，主要整理一些面试过程中常见的算法题 ； 第二个 `ECMAScript规范解析`，主要是想从ES规范的角度来解析JS代码的实际执行过程，只有真正懂得了底层干了什么，才能完全理解JS，而不需要记住很多零散的坑； 第三个 `JS开源库代码阅读`，通过阅读开源代码加深对JS的理解，同时积累大型项目开发的最佳实践。


> 本系列为 `JS算法系列`
> 1. [求两个已排序数组的中位数](#-1-求两个已排序数组的中位数)
> 2. [归并排序](#-2-归并排序)
> 3. [交换排序：冒泡排序/奇偶排序/快速排序](#-3-交换排序冒泡排序奇偶排序快速排序)
> 4. [插入排序：简单插入排序/希尔排序](#-4-插入排序简单插入排序希尔排序)
> 5. [选择排序：简单选择排序/堆排序](#-5-选择排序简单选择排序堆排序)

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

The median is (2 + 3) / 2 = 2.5
```

#### 分治法 Divide and Conquer

- 思路
    1. 假设两个数组共有len个元素，如果len为奇数，中位数为查找最终合并数组中的第 len/2+1 个元素；如果len为偶数，中位数为查找最终合并数组中的第 len/2 个 和 len/2+1 个元素。可以归结为TOP(k)问题, 如果解决了TOP(k)问题，那么求中位数不过是k取不同值的问题;
    2. 那么如何求TOP(k)问题呢？可以有以下两种方法

    方法一：取第一个数组的第p个元素，第二个数组的第q（保证 p+q===k，如果p确定了，q就随之确定了）个元素；如果第一个数组的第 p 个元素小于等于第二个数组的第 q 个元素，虽然我们不能确定第二个数组的第q个元素是不是我们要找的k元素，但是第一个数组的前p个元素一定不包含k元素（反证法：如果k元素为第一个数组第x1个元素，已知x1<=p，那么 x1+(q-1) <= p+q-1= k-1，此时x1对应的元素在最终合并的数组中最多为第k-1个元素，与我们的假设x1元素是K元素矛盾），因此可以抛弃第一个数组的前p个元素，形成一个较短的新数组；然后用新数组替换原先的第一个数组，再找这两个数组中的第k-p(更新k到k-p)个元素，依次递归。同理，如果第一个数组的第p个元素大于第二个数组的第q个元素，那么可以抛弃第二个数组的前q个元素。递归的终止条件包括两种情况：
    *  如果其中一个数组中的所有元素都被抛弃，直接返回另外一个数组的第k个元素；
    *  如果 k == 1， 直接返回第一个数组第一个元素和第二个数组第一个元素中的较小者；
    
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
>   NOTE1: JS中数字均为浮点数，因此需要使用 parseInt 取整<br>
>   NOTE2: findKth函数的代码中第一个数组的第p个元素的下标是pos1, 它是该数组中的第 pos1-start1+1 个元素，即第parseInt(k / 2)个元素；同理，第二个数组中的第q个元素为第二个数组中的第 parseInt(k / 2)个元素，此时p+q = 2 * parseInt(k / 2)， 当k为基数时 2 * parseInt(k / 2) 比 k 小1， 而我们的假设中p + q === k，之所以上述代码可以得到正确的结果是因为 k - 1 <= p + q <= k，只要p+q 不大于k，被淘汰掉的parseInt(k / 2)个元素中就不可能包含TOP(K)元素，只不过是有时候在一次递归过程中少淘汰一个元素而已。下面的代码我们严格按照 p + q === k来执行淘汰，也能得到正确的结果（仅修改了添加注释的两行代码）：

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



### _#_ 2. 归并排序

归并排序是建立在归并操作上的一种高效的排序算法，该算法是分治法（分而治之）的一个典型应用。

首先考虑下如何将两个有序数组合并，思路很简单：从头开始比较两个数组的对应元素，将较小的元素先插入到临时数组中，更新索引下标；如果其中一个数组为空，则另外一个数组中剩下的元素依次进入临时数组。

代码如下：
```js
function mergeArray(arr1, arr2, temp_arr) {
    var i = 0, j = 0;
    var len1 = arr1.length, len2 = arr2.length;

    while (i < len1 && j < len2) {
        if (arr1[i] <= arr2[j]) {
            temp_arr.push(arr1[i++]);
        } else {
            temp_arr.push(arr2[j++]);
        }
    }

    while (i < len1) {
        temp_arr.push(arr1[i++]);
    }
    while (j < len2) {
        temp_arr.push(arr2[j++]);
    }
    return temp_arr;
}
```

>   题外话：JS中Array.prototype.push(...items)插入一个元素与使用arr[arr.length] = elem 效率差不多，所不同的是 push 函数可以一次性插入多个元素，并且返回插入后数组的length；需要注意的是引擎会在底层对数组对象的存储及属性访问进行优化，存储稀疏数组的底层对象与存储一般数组的底层对象不同，因此需要避免先行设置数组的length属性，因为这样会使数组从一开始就是稀疏的，底层在存储的时候会采用不同的数据结构进行表示，降低了后续属性访问的效率

>   Array.prototype.push 是一个泛型函数，也就是说不要求一定在数组对象上调用该函数，因此规范中添加了``Perform ?Set(O, length, len, true)``操作来更新length属性（即使数组上添加新的元素会自动更新length属性）。

![](https://github.com/shadowkimi520/one_algorithm_one_day/raw/master/images/array_prototype_push.png)

有了上述归并两个有序数组的实现，我们再来看归并排序，它的基本思路就是将数组分为左右两个部分，如果这两个数组中的元素都是有序的，则可以通过归并它们生成最终的有序数组，从而达到排序的目的。

如何让左右两部分各自有序呢？可以将左右两个数组各自再分割成两组，依次下去，当分出来的子数组只有一个元素时，可以认为这个子数组已经有序了，然后再归并相邻的两个子数组。我们可以通过先递归地分割数组，再合并数组就实现了归并排序。

```js
function mergeArray(arr, first, mid, last, temp_arr) {
    var i = first, j = mid + 1, k = 0;

    while (i <= mid && j <= last) {
        if (arr[i] <= arr[j]) {
            temp_arr[k++] = arr[i++];
        } else {
            temp_arr[k++] = arr[j++];
        }
    }

    while (i <= mid) {
        temp_arr[k++] = arr[i++];
    }
    while (j <= last) {
        temp_arr[k++] = arr[j++];
    }

    for (i = 0; i < len; i++) {
        arr[first + i] = temp_arr[i];
    }
}

function mergeSort(arr, first, last, temp_arr) {
    if (first < last) {
        var mid = parseInt((first + last) / 2);
        mergeSort(arr, first, mid, temp_arr);
        mergeSort(arr, mid + 1, last, temp_arr);
        mergeArray(arr, first, mid, last, temp_arr);
    }
}

// mergeSort(arr, 0, arr.length - 1, temp_arr) 调用方式
```
>   总结：通过传入一个全局的temp_arr来减少每次归并过程中局部数组的需求，避免不必要的对象分配及垃圾回收，提高算法效率；另外由于临时数组中归并后的数据会复制到原数组对应位置当中去，因此 temp_arr的下标是从0开始复用的，我们也可以改成`temp_arr[first + k++] = arr[i++];` 不进行复用，此时临时数组的下标与原数组是对应的。

在所有的排序算法中，归并排序的效率是比较高的。如果数组长度为N，将数组分割成长度为1的子数组需要logN步，中间针对各个长度的子数组归并的时间复杂度为O(N)，所以归并排序的时间复杂度为O(NlogN)；因为需要临时数组来辅助归并过程，所以归并排序的空间复杂度为O(N)。*另外，归并排序是稳定的排序算法，在最差情况和最优情况下的时间复杂度均为O(NlogN)。*


### _#_ 3. 交换排序：冒泡排序/奇偶排序/快速排序

>   暂不考虑JS里面的稀疏数组，如果为稀疏数组，先遍历一遍把所有的稀疏元素移动到数组的最右边，然后开始执行原来的排序算法。

冒泡排序比较简单，主要思想是通过相邻元素的两两比较，把最大的元素一步一步交换到最右边的位置，此时最大的元素已经处于排序后的正确位置，排除最大的元素并继续对剩余元素应用冒泡排序，直到剩下唯一的元素。

标准的冒泡排序实现代码如下（非递归版本），也可以添加一个flag变量表示当前遍历是否有交换元素，如果没有则表示已经有序，所以可以提前退出外层循环：
```js
function bubbleSort(arr) {
    var len = arr.length;
    for (var minIndex = 0; minIndex < len - 1; minIndex++) {
        for (var tempIndex = len - 1; tempIndex > minIndex; tempIndex--) {
            if ( arr[tempIndex] < arr[tempIndex - 1]) {
                [arr[tempIndex - 1], arr[tempIndex]] = [arr[tempIndex], arr[tempIndex - 1]];
            }
        }
    }
}

// 添加flag优化版本
function bubbleSort(arr) {
    var len = arr.length, flag = true;
    for (var minIndex = 0; minIndex < len - 1 && flag; minIndex++) {
        flag = false;
        for (var tempIndex = len - 1; tempIndex > minIndex; tempIndex--) {
            if (arr[tempIndex] < arr[tempIndex - 1]) {
                [arr[tempIndex - 1], arr[tempIndex]] = [arr[tempIndex], arr[tempIndex - 1]];
                flag = true;
            }
        }
    }
}
```

实际上冒泡排序也可以不用相邻元素来进行两两比较，只要保证在每一趟遍历之后最小值到达正确位置就行，下面的代码在遍历过程中均和数组的起始元素比较，每一趟得到一个当前的最小值
```js
function bubbleSort(arr) {
    var len = arr.length;
    for (var minIndex = 0; minIndex < len - 1; minIndex++) {
        for (var tempIndex = minIndex + 1; temIndex < len; tempIndex++) {
            if (arr[tempIndex] < arr[minIndex]) {
                [arr[minIndex], arr[tempIndex]] = [arr[tempIndex], arr[minIndex]];
            }
        }
    }
}
```

从左到右的冒泡算法（实际上是沉底排序，而不是真正意义上的冒泡排序，冒泡排序保证最小的元素先调整到位）实现代码如下（非递归版本）：
```js
function bubbleSort(arr) {
    for (var maxIndex = arr.length - 1; maxIndex > 0; maxIndex--) {
        for (var tempIndex = 0; tempIndex < maxIndex; tempIndex++) {
            if (arr[tempIndex] > arr[tempIndex + 1]) {
                [arr[tempIndex], arr[tempIndex + 1]] = [arr[tempIndex + 1], arr[tempIndex]]; 
            }
        }
    }
}
```

由于冒泡排序的“[乌龟问题](https://blog.csdn.net/lemon_tree12138/article/details/50591859)”，又衍生出了一种双向冒泡排序，主要思想是：奇数趟冒泡，偶数趟沉底。（代码如下）
>   乌龟问题是说：假设我们需要将序列A按照升序序列排序。序列中的较小的数字又大量存在于序列的尾部，冒泡排序会让小数字向前移动得很缓慢。

```js
function bubbleSort(arr) {
    var len = arr.length;
    var startIndex = 0, endIndex = len - 1, tempIndex = 0;
    while (startIndex < endIndex) {
        for (tempIndex = endIndex; tempIndex > startIndex; tempIndex--) {
            if (arr[tempIndex] < arr[tempIndex - 1]) {
                [arr[tempIndex - 1], arr[tempIndex]] = [arr[tempIndex], arr[tempIndex - 1]];
            }
        }
        startIndex++;

        for (tempIndex = startIndex; tempIndex < endIndex; tempIndex++) {
            if (arr[tempIndex] > arr[tempIndex + 1]) {
                [arr[tempIndex], arr[tempIndex + 1]] = [arr[tempIndex + 1], arr[tempIndex]];
            }
        }
        endIndex--;
    }
}
```

> 快速排序是对冒泡排序的一种改进，均属于交换排序的算法。（交换排序包括：冒泡排序/奇偶排序/快速排序）

&nbsp;&nbsp;&nbsp;&nbsp;快速排序的基本思想是：通过一趟排序将要排序的数组分割成两个独立的部分，其中一部分的的所有数据都比另外一部分的数据小；然后再按照此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，从而使得最终的数组有序。快速排序作为一种交换排序，其交换方式主要有两种：标准交换法 和 两头交换法。

#### 标准交换法步骤
> 1. 从待排序数组中选择一个合适的元素elem作为排序基准（常规做法是选取待排序数组的第一个元素；也可以随机选择待排序数组的某个元素，将其与第一个元素交换，然后再按标准流程排序，这样可以避免当待排序数组部分有序的时候算法效率降低，交换后可以确保刚开始的时候数组的第一个位置是一个_被挖出的坑_，需要被填充）；
> 2. 执行一趟排序过程，将待排序数组中所有比基准元素小的元素交换到左边，所有比基准元素大的元素交换到右边，此时基准元素在待排序数组中交换到了正确的位置，而且其左右被分割成了两个待排序的子数组sub1和sub2;
> 3. 判断sub1和sub2的元素个数，如果只有一个元素，则该子数组已经有序；否则分别对sub1/sub2递归执行快速排序；

#### 代码

```js
/*
 * 功能：交换并返回分界点（标准交换法）
 * 
 * @param arr
 *      待排序数组
 * @param startIndex
 *      开始位置
 * @param endIndex
 *      结束位置
 * @return
 *      分界点
 */
function partition(arr, startIndex, endIndex) {
    var base = arr[startIndex];
    var leftIndex = startIndex, rightIndex = endIndex;
    while (leftIndex < rightIndex) {
        while (leftIndex < rightIndex && arr[rightIndex] >= base) {
            rightIndex--;
        }
        arr[leftIndex] = arr[rightIndex];
        while (leftIndex < rightIndex && arr[leftIndex] <= base) {
            leftIndex++;
        }
        arr[rightIndex] = arr[leftIndex];
    }

    arr[leftIndex] = base;
    return leftIndex;
}
/*
 * 排序的核心算法
 * 
 * @param arr
 *      待排序数组
 * @param startIndex
 *      开始位置
 * @param endIndex
 *      结束位置
 */
function quickSort(arr, startIndex, endIndex) {
    if (startIndex >= endIndex) {
        return ;
    }
    var boundary = partition(arr, startIndex, endIndex);
    quickSort(arr, startIndex, boundary - 1);
    quickSort(arr, boundary + 1, endIndex);
}
```

#### 两头交换法主要思想
> 两头交换法先从左边开始找到大于基准值的那个数，再从右边找到小于基准值的那个数，将两个数交换(这样比基准值小的都在左边，比基准值大的都在右边)，而两头交换法又分为_官方版本_和_基准不参与交换版本_， _官方版本_的主要思想是基准元素也参与交换过程，一趟排序过后，可以确保左边子数组中的元素都小于基准值，右边子数组中的元素都大于等于基准值，此时基准值在数组中不一定处于正确的位置，再递归排序左右两个子数组；_基准不参与交换版本_的主要思想和标准交换法类似，也是保证一趟排序后基准值到达最终的正确位置，只是在交换的时候采用两头交换的方式加快算法的处理速度。 

两头交换法-官方版
```js
function partition(arr, startIndex, endIndex) {
    // 官方版本中基准值参与交换过程，所以基准值可以取任意位置的元素，此处我们取中位数
    var base = arr[(startIndex + endIndex) >> 1];  
    var leftIndex = startIndex, rightIndex = endIndex;

    while (leftIndex <= rightIndex) {
        while (arr[leftIndex] < base) leftIndex++;
        while (arr[rightIndex] > base) rightIndex--;

        if (leftIndex <= rightIndex) {
            [arr[leftIndex], arr[rightIndex]] = [arr[rightIndex], arr[leftIndex]]; // 交换元素
            leftIndex++;
            rightIndex--;
        }
    }
    return [leftIndex, rightIndex];
}

function quickSort(arr, startIndex, endIndex) {
    if (startIndex >= endIndex) {
        return ;
    }
    var [leftIndex, rightIndex] = partition(arr, startIndex, endIndex);
    quickSort(arr, startIndex, rightIndex);
    quickSort(arr, leftIndex, endIndex);
}
```

两头交换法-基准不参与交换版
```js
function partition(arr, startIndex, endIndex) {
    // 基准不参与交换版本中基准值只能取第一个元素或者最后一个元素，此处我们取第一个元素，取最后一个元素的代码类似，需要确保填充最后一个元素“坑”的元素是不小于基准值的第一个元素
    var base = arr[startIndex]; 
    var leftIndex = startIndex + 1, rightIndex = endIndex;
    
    while (leftIndex <= rightIndex) {
        while (leftIndex <= endIndex && arr[leftIndex] < base) leftIndex++;
        while (arr[rightIndex] > base) rightIndex--;

        if (leftIndex <= rightIndex) {
            [arr[leftIndex], arr[rightIndex]] = [arr[rightIndex], arr[leftIndex]]; // 交换元素
            leftIndex++;
            rightIndex--;
        }
    }
    arr[startIndex] = arr[rightIndex];
    arr[rightIndex] = base;
    return rightIndex;
}

function quickSort(arr, startIndex, endIndex) {
    if (startIndex >= endIndex) {
        return ;
    }
    var boundary = partition(arr, startIndex, endIndex);
    quickSort(arr, startIndex, boundary - 1);
    quickSort(arr, boundary + 1, endIndex);
}
```

### 4. _#_ 插入排序：简单插入排序/希尔排序
插入排序顾名思义：将一个元素插入到一个有序数组中，使得插入该元素后的数组依然有序。

> 简单插入排序的思想：第一个元素形成的子数组已然有序，第二个元素开始往后选择哨兵元素，依次将哨兵元素插入到前面子数组的正确位置，使得包含哨兵元素的子数组也是有序的，该算法是稳定的。
```js
function insertSort(arr) {
    var endIndex = arr.length - 1, tempIndex;
    for (var startIndex = 1; startIndex <= endIndex; startIndex++) {
        var sentinel = arr[startIndex];
        for (tempIndex = startIndex - 1; tempIndex >= 0; tempIndex--) {
           if (arr[tempIndex] > sentinel) {
               arr[tempIndex + 1] = arr[tempIndex];
           } else {
               break; // 最大的元素都比哨兵元素小，前面的不需要进行比较
           }
        }
        arr[tempIndex + 1] = sentinel;
    }
}
```

> 简单插入排序在数组已经有序或者部分有序的情况下效率比较高，shell排序正是利用了这一点。希尔排序的主要思想是：把一个数组中的所有元素按照步长（也称增量）分为几部分，先对几个小部分的元素各自执行简单插入排序，让数组的元素大概有序，最后在所有元素上执行一遍简单插入排序。希尔排序是不稳定的排序算法。
```js
function shellSort(arr) {
    var len = arr.length; d = len >> 1;
    while (d > 0) {
        for (var startIndex = d; startIndex < len; startIndex++) {
            var sentinel = arr[startIndex];
            for (var tempIndex = startIndex - d; tempIndex >= 0; tempIndex -= d) {
                if (arr[tempIndex] > sentinel) {
                    arr[tempIndex + d] = arr[tempIndex];
                } else {
                    break;
                }
            }
            arr[tempIndex + d] = sentinel;
        }
        d >>= 1;
    }
}
```

### 5. _#_ 选择排序：简单选择排序/堆排序
> 选择排序顾名思义就是每次从待排序数组中找出当前最小的元素使其处于正确的位置。

简单选择排序的思路比较简单，起初整个数组都是无须的，从下标0开始选择当前最小的元素置于0位置；在第一趟排序之后，位置0的元素已经是最小的了，继续从下标1开始选择剩下数组中的最小值并置于位置1；依次执行，直到未排序的数组中只剩下最后一个元素，此时它就是最小值，排序完成。
```js
function selectionSort(arr) {
    var len = arr.length;
    for (var startIndex = 0; startIndex < len - 1; startIndex++) {
        var min = arr[startIndex], minIndex = startIndex;
        for (var tempIndex = startIndex + 1; tempIndex < len; tempIndex++) {
            if (arr[tempIndex] < min) {
                min = arr[tempIndex];
                minIndex = tempIndex;
            }
        }
        if (minIndex !== startIndex) {
            [arr[startIndex], arr[minIndex]] = [arr[minIndex], arr[startIndex]];
        }
    }
}
```

<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.12.0/styles/monokai.min.css">

<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.12.0/highlight.min.js"></script>


<script>
hljs.configure({
  languages: ['cs']
})
hljs.initHighlightingOnLoad();
</script>