---
title: leetcode-problems
tags:
  - 算法
categories:
  - 九章算法
date: 2019-07-21 16:41:30
---


# Leetcode Problems #

leetcode 一些题目答案的整理。

### Permutations ###

[leedcode地址](https://leetcode-cn.com/problems/permutations/)

```
var permute = function(nums) {
    var result = [];
    if(nums.length === 0) {
        return result;
    }
    fun(nums, [], result);
    return result;
};

var fun = function(nums, current, result) {
    for(var i = 0; i < nums.length; i++) {
        current.push(nums[i]);
        const numsCopy = nums.concat();
        numsCopy.splice(i, 1);
        if (numsCopy.length === 0) {
            result.push(current.concat());
        } else {
            fun(numsCopy, current, result);
        }
        current.pop(nums[i]); // 回溯。
    }
}
```

### N queens ###

&emsp;&emsp;经典的n皇后问题，主要解题思路是广度优先搜索和回溯。

[leedcode地址](https://leetcode-cn.com/problems/n-queens/)

```
var solveNQueens = function(n) {
    var result = [];
    var chessboard = [];
    // 初始化棋盘
    for (var x = 0; x < n; x++) {
        chessboard[x] = [];
        for (var y = 0; y < n; y++) {
            chessboard[x] += ".";
        }
    }
    backtrack(0, [], result, chessboard, n);
    return result
};

function backtrack(row, queenCoordinates, results, chessboard, n) { // 回溯的方法。
    for(var i = 0; i < n; i++) {
        if (!isConflict(queenCoordinates, row, i)) {
            queenCoordinates.push({ x: row, y: i});
            if (row + 1 === n) {
                const result = chessboard.concat();
                for (var coordinate of queenCoordinates) {
                    result[coordinate.x] = result[coordinate.x].slice(0, coordinate.y) + "Q" + result[coordinate.x].slice(coordinate.y + 1);
                }
                results.push(result);
            } else {
                backtrack(row + 1, queenCoordinates.concat(), results, chessboard, n);
            }
            queenCoordinates.pop();
        }
    }
}

function isConflict(queenCoordinates, x, y) { // 判断棋子是否冲突了。
    for(var i = 0; i < queenCoordinates.length; i++) {
        const coordinate = queenCoordinates[i];
        if (coordinate.x === x || coordinate.y === y ||
            (coordinate.y - coordinate.x) === (y - x) ||
            (coordinate.y + coordinate.x) === (x + y)) {
                return true;
        }
    }
    return false;
}
```

### Single number ###

&emsp;&emsp;使用了异或的特性。

[leedcode地址](https://leetcode-cn.com/problems/single-number/)

```
var singleNumber = function(nums) {
    var result = 0;
    for(var i = 0; i < nums.length; i++) {
        result = result ^ nums[i]
    }
    return result;
};
```

### Majority element ###

&emsp;&emsp;不同元素抵消，剩余的元素一定是 Majority element;

[leedcode地址](https://leetcode-cn.com/problems/majority-element/)

```
var majorityElement = function(nums) {
    var array = [];
    var head;
    for (var i = 0; i < nums.length; i++) {
        if (nums[i] != head && array.length !== 0) {
            array.pop();
        } else {
            array.push(nums[i]);
            head = nums[i];
        }
    }
    return array.pop();
};
```