---
title: binary-search
tags:
  - 算法
categories:
  - 九章算法
date: 2019-05-30 14:37:51
---


# Binary Search #

&emsp;&emsp;这是一个关于二分查找相关的leedcode题的整理。

### First bad version ###

&emsp;&emsp;很有趣的一道题，实际开发中我曾经遇到过类似的问题，当时傻傻的人工二分查找，感觉之后不会那么傻了，完全可以使用这种方式。典型的二分查找首个或者末个元素的题。

[leedcode地址](https://leetcode-cn.com/problems/first-bad-version/)

```
var solution = function(isBadVersion) {
    /**
     * @param {integer} n Total versions
     * @return {integer} The first bad version
     */
    return function(n) {
        var start = 1;
        var end = n;
        while(start + 1 < end) {
            var mid = start + Math.floor((end - start) / 2);
            if (isBadVersion(mid)) {
                end = mid;
            } else {
                start = mid;
            }
        }
        if (isBadVersion(start)) {
            return start;
        }
        if (isBadVersion(end)) {
            return end;
        }
        return -1;
    };
};
```

### Search first&last positon ###

[leedcode地址](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

```
var searchRange = function(nums, target) {
    if (nums.length === 0) {
        return [-1, -1];
    }
    var start = 0;
    var end = nums.length - 1;
    var firstIndex, endIndex, mid;

    while(start + 1 < end) {
        mid = start + Math.floor((end - start) / 2);
        if (nums[mid] >= target) {
            end = mid;
        } else {
            start = mid;
        }
    }
    if (nums[start] === target) {
        firstIndex = start;
    } else if (nums[end] === target) {
        firstIndex = end;
    } else {
        return [-1, -1];
    }

    start = firstIndex;
    end = nums.length - 1;

    while(start + 1 < end) {
        mid = start + Math.floor((end - start) / 2);
        if (nums[mid] <= target) {
            start = mid;
        } else {
            end = mid;
        }
    }
    if (nums[end] === target) {
        endIndex = end;
    } else if (nums[start] === target) {
        endIndex = start;
    }
    return [firstIndex, endIndex];
};
```

### Search insert position ###

&emsp;&emsp;二分查找第一个大于或等于target的index 即可。

[leedcode地址](https://leetcode-cn.com/problems/search-insert-position/)

```
var searchInsert = function(nums, target) {
    if (nums.length === 0) {
        return 0;
    }

    var start = 0;
    var end = nums.length - 1;

    while(start + 1 < end) {
        var mid = start + Math.floor((end - start) / 2);
        if (nums[mid] >= target) {
            end = mid;
        } else {
            start = mid;
        }
    }
    if (nums[start] >= target) {
        return start;
    }
    if (nums[end] >= target) {
        return end;
    }
    return nums.length;
};
```

### Search a 2D matrix ###

&emsp;&emsp;可以将二维矩阵理解为一个特殊数据处理即可。

[leedcode地址](https://leetcode-cn.com/problems/search-a-2d-matrix/)

```
var searchMatrix = function(matrix, target) {
    var m = matrix.length;
    if (m === 0) {
        return false;
    }
    var n = matrix[0].length;
    if (n === 0) {
        return false;
    }
    var start = 0;
    var end = m * n - 1;
    while (start + 1 < end) {
        var mid = start + Math.floor((end - start) / 2);
        if (getMatrixValue(matrix, mid, n) === target) {
            return true;
        }
        if (getMatrixValue(matrix, mid, n) > target) {
            end = mid;
        } else {
            start = mid;
        }
    }
    if (getMatrixValue(matrix, start, n) === target || getMatrixValue(matrix, end, n) === target) {
        return true;
    }
    return false;
};

function getMatrixValue(matrix, index, n) {
    return matrix[Math.floor(index / n)][index % n];
}
```

### Search a 2D Matrix II ###

&emsp;&emsp;思路是从左下角开始找起，如果元素大于target，上移，如果小于target，右移，相等直接返回true。

[leedcode地址](https://leetcode-cn.com/problems/search-a-2d-matrix-ii/)

```
var searchMatrix = function(matrix, target) {
    var m = matrix.length;
    if (m === 0) {
        return false;
    }
    var n = matrix[0].length;
    if (n === 0) {
        return false;
    }

    var row = m - 1;
    var col = 0;
    while (row >=0 && col < n) {
        if (matrix[row][col] === target) {
            return true;
        } else if (matrix[row][col] > target) {
            row--;
        } else {
            col++;
        }
    }
    return false;
};
```

### Find minimum in rotated sorted array ###

[leedcode地址](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)

```
var findMin = function(nums) {
    if (nums.length === 0) {
        return null;
    }
    var start = 0;
    var end = nums.length - 1;
    while (start + 1 < end) {
        var mid = start + Math.floor((end - start) / 2);
        if (nums[mid] < nums[end]) {
            end = mid;
        } else {
            start = mid;
        }
    }
    if (nums[start] <= nums[end]) {
        return nums[start];
    } else {
        return nums[end];
    }
};
```

### Find minimum in rotated sorted array II ###

&emsp;&emsp;这个需要通过黑盒测试证明时间复杂度是O(n)，比如针对[1, 1, 0, 1, 1, ,1 ,1]就无法使用二分法查找最小值。

[leedcode地址](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array-ii/)

### Search in rotated sorted array ###

[leedcode地址](https://leetcode-cn.com/problems/search-in-rotated-sorted-array)

```
var search = function(nums, target) {
    var start = 0;
    var end = nums.length - 1;
    while (start + 1 < end) {
        var mid = start + Math.floor((end - start) / 2);
        if (nums[mid] === target) {
            return mid;
        }
        if ((nums[start] <= target && target < nums[mid]) || (nums[start] >= nums[mid] && (nums[start] <= target || target < nums[mid]))) {
            end = mid;
        } else if ((nums[mid] < target && target <= nums[end] ) || (nums[mid] >= nums[end] && (nums[mid] < target || target <= nums[end]))) {
            start = mid;
        } else {
            start = end;
        }
    }
    if (nums[start] === target) {
        return start;
    }
    if (nums[end] === target) {
        return end;
    }
    return -1;
};
```

### Search in rotated sorted array II ###

&emsp;&emsp;这个也可以依据黑盒算法证明无法通过二分查找计算结果。

[leedcode地址](https://leetcode-cn.com/problems/search-in-rotated-sorted-array-ii)