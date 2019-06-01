---
title: sorted-array
tags:
  - 算法
categories:
  - 九章算法
date: 2019-06-01 10:06:42
---


# Sorted Array #

### Merge sorted array ###

&emsp;&emsp;主要思路：倒序从数组的尾部开始合并。

[leedcode地址](https://leetcode-cn.com/problems/merge-sorted-array/)

```
var merge = function(nums1, m, nums2, n) {
  var latstIndex = m + n - 1;
  var nums1LastIndex = m - 1;
  var nums2LastIndex = n -1;
  while(latstIndex >= 0 && nums2LastIndex >=0) {
    if (nums1LastIndex === -1 || nums2[nums2LastIndex] >= nums1[nums1LastIndex]) {
      nums1[latstIndex] = nums2[nums2LastIndex];
      nums2LastIndex--;
    } else {
      nums1[latstIndex] = nums1[nums1LastIndex];
      nums1LastIndex--;
    }
    latstIndex--;
  }
  return nums1;
};
```

### Recover rotated dorted array ###

&emsp;&emsp;Given a rotated sorted array, recover it to sorted array in-place

Example

[4, 5, 1, 2, 3] -> [1, 2, 3, 4, 5]

使用O(1)的额外空间

三步翻转法。

[4, 5, 1, 2, 3] -> [3, 2, 1, 5, 4] -> [1, 2, 3, 5, 4] -> [1, 2, 3, 4, 5]

### Median of two sorted arrays ###

[leedcode地址](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)

解法一：

&emsp;&emsp;这个方法是我做这道题时候最先想到的方法。时间复杂度是O(min(m, n))并不能达到O(log(n + m))。
```
var findMedianSortedArrays = function(nums1, nums2) {
    var length = nums1.length + nums2.length;
    var half = Math.floor(length / 2);
    var isJi = length % 2;
    if (nums1.length > nums2.length) {
        var temp = nums1;
        nums1 = nums2;
        nums2 = temp;
    }
    if (nums1.length === 0) {
        if (isJi) {
            return nums2[half];
        } else {
            return (nums2[half - 1] + nums2[half]) / 2;
        }
    }
    for (var i = -1; i < nums1.length; i ++) {
        if (i === -1) {
            if (nums1[0] >= nums2[half - 1]) {
                if (isJi) {
                    return nums2[half] && (nums2[half] < nums1[0]) ? nums2[half] : nums1[0];
                } else {
                    return (nums2[half - 1] + (nums2[half] && (nums2[half] < nums1[0]) ? nums2[half] : nums1[0])) / 2;
                }
            }
        } else if (i == nums1.length - 1) {
            if (nums1[nums1.length - 1] <= nums2[half - i - 1]) {
                if (isJi) {
                    return nums2[half - i - 1];
                } else {
                    return (nums2[half - i -1] + (nums2[half - i - 2] && (nums2[half - i - 2] > nums1[i]) ? nums2[half - i - 2] : nums1[i])) / 2;
                }
            }
        } else {
            var a1 = nums1[i];
            var a2 = nums1[i + 1];
            var b1 = nums2[half - i - 2];
            var b2 = nums2[half - i - 1];
            if (a1 <= b2 && b1 <= a2) {
                if (isJi) {
                    return a2 && a2 < b2 ? a2 : b2;
                } else {
                    return ((a1 > b1 ? a1 : b1) + (a2 && a2 < b2 ? a2 : b2)) / 2;
                }
        }
        }
    }
};
```

解法二：

&emsp;&emsp;最终转换成了一个查到第k大的问题。

```
var findMedianSortedArrays = function(nums1, nums2) {
    var len = nums1.length + nums2.length;
    var halfLen = Math.floor(len / 2);
    if (len % 2 === 0) {
        return (findKth(nums1, 0, nums2, 0, halfLen) + findKth(nums1, 0, nums2, 0, halfLen + 1)) / 2;
    } else {
        return findKth(nums1, 0, nums2, 0, halfLen + 1);
    }
};

function findKth(nums1, start1, nums2, start2, k) {
    if(start1 >= nums1.length) {
        return nums2[start2 + k -1];
    }
    if (start2 >= nums2.length) {
        return nums1[start1 + k - 1];
    }
    if (k === 1) {
        return Math.min(nums1[start1], nums2[start2]);
    }
    var halfK = Math.floor(k / 2);
    // 如果加halfK超过数据长度，表示half一定存在于另一个数组中，直接取Infinity表示最大值。
    var key1 = start1 + halfK - 1 < nums1.length ? nums1[start1 + halfK - 1] : Infinity;
    var key2 = start2 + halfK - 1 < nums2.length ? nums2[start2 + halfK - 1] : Infinity;
    if (key1 < key2) {
        return findKth(nums1, start1 + halfK, nums2, start2, k - halfK);
    } else {
        return findKth(nums1, start1, nums2, start2 + halfK, k - halfK);
    }
}
```