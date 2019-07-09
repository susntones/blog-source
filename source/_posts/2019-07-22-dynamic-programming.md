---
title: dynamic-programming
tags:
  - 算法
categories:
  - 九章算法
date: 2019-07-22 11:24:24
---


# Dynamic Programming #

处理DP问题的两种方式

* 记忆化搜索
* 循环，包括自底向上和自顶向下两种方式。

DP常见的问题类型

* matrix dp
* sequence
* two sequence dp
* backpack

动态规划的4点要素

* 状态 State

  存储小规模问题的结果。
* 方程 Function

  状态之间的联系，怎样通过小的状态，来算大的状态。
* 初始化 Intialization

  最极限的小状态
* 答案 Answer

  最终状态

## Matrix dp ##

### Triangle ###

这道题有多种解决方法：

* 递归
    * 遍历
    * 分治
* DP
    * 记忆化搜索
    * 循环（自底向上 + 自定向下）

[leedcode地址](https://leetcode-cn.com/problems/triangle/)

分治的方法
```
var minimumTotal = function(triangle) {
  return getMinPath(0, 0, triangle);
};

function getMinPath(x, y, triangle) {
  if (x === triangle.length - 1) {
    return triangle[x][y];
  }
  var left = getMinPath(x + 1, y, triangle);
  var right = getMinPath(x + 1, y + 1, triangle);
  return Math.min(left, right) + triangle[x][y];
}
```

DP 循环（自底向上）
```
var minimumTotal = function(triangle) {
    var length = triangle.length;
    var array = [];
    for (var i = 0; i < length; i++) {
        for(var j = 0; j < length - i; j++) {
            if (i === 0) {
                array[j] = triangle[length - i - 1][j];
            } else {
                array[j] = Math.min(array[j], array[j + 1]) + triangle[length - i - 1][j];
            }
        }
    }
    return array[0];
};
```

### Unique paths ###

[leedcode地址](https://leetcode-cn.com/problems/unique-paths/)

```
var uniquePaths = function(m, n) {
    var paths = [];
    for (var j = 0; j < n; j++) {
        paths[j] = [];
        paths[j][0] = 1;
    }
    for (var i = 0; i < m; i++) {
        paths[0][i] = 1;
    }
    for (var height = 1; height < n; height++) {
        for (var weight = 1; weight < m; weight ++) {
            paths[height][weight] = paths[height - 1][weight] + paths[height][weight - 1];
        }
    }
    return paths[n - 1][m - 1];
};
```

### Unique paths II ###

[leedcode地址](https://leetcode-cn.com/problems/unique-paths-ii/)

```
var uniquePathsWithObstacles = function(obstacleGrid) {
    var m = obstacleGrid[0].length;
    var n = obstacleGrid.length;
    var paths = [];
    for (var j = 0; j < n; j++) {
        paths[j] = [];
        if (obstacleGrid[j][0] === 1 || (j > 0 && paths[j - 1][0] === 0)) {
            paths[j][0] = 0;
        } else {
            paths[j][0] = 1;
        }
    }
    for (var i = 0; i < m; i++) {
        if (obstacleGrid[0][i] === 1 || (i > 0 && paths[0][i - 1] === 0)) {
            paths[0][i] = 0;
        } else {
            paths[0][i] = 1;
        }
    }
    for (var height = 1; height < n; height++) {
        for (var weight = 1; weight < m; weight ++) {
            if (obstacleGrid[height][weight] === 1) {
                paths[height][weight] = 0;
            } else {
                paths[height][weight] = paths[height - 1][weight] + paths[height][weight - 1];
            }
        }
    }
    return paths[n - 1][m - 1];
};
```

## Sequence dp ##

### Climbing stairs ###

[leedcode地址](https://leetcode-cn.com/problems/climbing-stairs/)

```
var climbStairs = function(n) {
    if (n === 1) {
        return 1;
    }
    var nums = [];
    nums[0] = 1;
    nums[1] = 1;
    for (var i = 2; i <= n; i++) {
        nums[i] = nums[i - 2] + nums[i - 1];
    }
    return nums[n];
};
```

### Jump game ###

[leedcode地址](https://leetcode-cn.com/problems/jump-game/)

```
var canJump = function(nums) {
    var length = nums.length;
    var canJumpArray = [];
    canJumpArray[0] = true;
    for (var i = 1; i < length; i++) {
        canJumpArray[i] = false;
        for (var j = i - 1; j >= 0; j--) {
            if (canJumpArray[j] && nums[j] >= i - j) {
                canJumpArray[i] = true;
                break;
            }
        }
    }
    return canJumpArray[length - 1];
}
```

### Jump game II ###

[leedcode地址](https://leetcode-cn.com/problems/jump-game-ii/)

DP的解法
```
var jump = function(nums) {
    var length = nums.length;
    var jumpNums = [];
    jumpNums[0] = 0;
    for (var i = 1; i < length; i++) {
        jumpNums[i] = Infinity;
        for (var j = i - 1; j >= 0; j--) {
            if (jumpNums[j] !== Infinity && nums[j] >= i - j) {
                jumpNums[i] = Math.min(jumpNums[j] + 1, jumpNums[i])
            }
        }
    }
    return jumpNums[length - 1];
};
```

贪心的方法

    从一个位置跳到它能跳到的最远位置之间的都只需要一步!所以,如果一开始都能跳到,后面再跳到的肯定步数要变多!
```
var jump = function(nums) {
    if (nums.length <= 1) {
        return 0;
    }
    var length = nums.length;
    var result = [];
    result[0] = 0;
    for (var i = 0; i < length; i ++) {
        for(var j = nums[i]; j > 0; j--) {
            if (i + j >= length - 1) {
                return result[i] + 1;
            } else if (result[i + j] === undefined) {
                result[i + j] = result[i] + 1;
            } else {
                break;
            }
        }
    }
    return result[length - 1];
};
```

### Palindrome partitioning ###

[leedcode地址](https://leetcode-cn.com/problems/palindrome-partitioning/)

```
var partition = function(s) {
    var buffer = new Buffer(s);
    var length = buffer.length;
    var array = [];
    array[0] = [[]];
    for (var i = 0; i < length; i++) {
        array[i + 1] = [];
        for (var j = i; j >=0; j--) {
            var isPal = isPalindrome(buffer, j, i);
            var str = buffer.slice(j, i + 1).toString();
            if (isPal) {
                for(var iterm of array[j]) {
                    var clone = iterm.concat();
                    clone.push(str)
                    array[i + 1].push(clone);
                }
            }
        }
    }
    return array[length];
};

function isPalindrome(buffer, start, end) {
    var result = true;
    while (start < end) {
        if (buffer[start] !== buffer[end]) {
            return false
        }
        start++;
        end--;
    }
    return result;
}
```

### Palindrome partitioning II ###

[leedcode地址](https://leetcode-cn.com/problems/palindrome-partitioning-ii/)

```
var minCut = function(s) {
    var buffer = new Buffer(s);
    var length = buffer.length;
    var array = [];
    array[0] = [0];
    for (var i = 0; i < length; i++) {
        array[i + 1] = Infinity;
        for (var j = i; j >=0; j--) {
            var isPal = isPalindrome(buffer, j, i);
            if (isPal) {
                array[i + 1] = Math.min(array[i + 1], array[j] + 1)
            }
        }
    }
    return array[length] - 1;
};

function isPalindrome(buffer, start, end) {
    var result = true;
    while (start < end) {
        if (buffer[start] !== buffer[end]) {
            return false
        }
        start++;
        end--;
    }
    return result;
}
```

### Word break ###

[leedcode地址](https://leetcode-cn.com/problems/word-break/)

```
var wordBreak = function(s, wordDict) {
    var minLength = Infinity;
    var maxLength = 0;
    wordDict.forEach((word) => {
        minLength = Math.min(minLength, word.length);
        maxLength = Math.max(maxLength, word.length);
    })
    if (minLength > s.length) {
        return false;
    }
    var array = [];
    for (var i = minLength - 1; i < s.length; i++) {
        for (var j = minLength; j <= maxLength; j++) {
            if (wordDict.includes(s.slice(i - j + 1, i + 1)) && (i - j < 0 || array[i - j])) {
                array[i] = true;
                break;
            }
        }
    }
    return array[s.length - 1] ? true : false;
};
```

### Longest increasing subsequence（LIS） ###

[leedcode地址](https://leetcode-cn.com/problems/longest-increasing-subsequence/)

```
var lengthOfLIS = function(nums) {
    if (nums.length === 0) {
        return 0;
    }
    var dp = [];
    var max = 0;
    for (var i = 0; i < nums.length; i++) {
        dp[i] = 1;
        for (var j = 0; j < i; j++) {
            if (nums[i] > nums[j]) {
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
        max = Math.max(max, dp[i]);
    }
    return max;
};
```

### Edit distance ###

[leedcode地址](https://leetcode-cn.com/problems/edit-distance/)

```
var minDistance = function(word1, word2) {
    var m = word1.length;
    var n = word2.length;
    var dp = [];
    for (var i = 0; i <= m; i++) {
        dp[i] = [];
        dp[i][0] = i;
    }
    for (i = 0; i <= n; i++) {
        dp[0][i] = i;
    }
    for(i = 1; i <= m; i++) {
        for(j = 1; j <= n; j++) {
            if(word1.charAt(i - 1) == word2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1];
            } else {
                dp[i][j] = 1 + Math.min(dp[i - 1][j - 1], Math.min(dp[i - 1][j], dp[i][j - 1]));
            }
        }
    }

    return dp[m][n];
};
```

## 背包问题 ##

[背包问题九讲](https://github.com/tianyicui/pack)

### Partition equal subset sum ###

&emsp;&emsp;01背包问题

[leedcode地址](https://leetcode-cn.com/problems/partition-equal-subset-sum/)

```
var canPartition = function(nums) {
    var sum = 0;
    for (var i = 0; i < nums.length; i++) {
        sum += nums[i];
    }
    if (sum % 2) {
        return false;
    }
    var mid = sum / 2;
    var dp = [];
    for (i = 0; i <= nums.length; i++) {
        dp[i] = [];
        dp[i][0] = true;
    }
    for (i = 1; i <= mid; i++) {
        dp[0][i] = false;
    }
    for (i = 1; i <= nums.length; i++) {
        for (var j = 1; j <= mid; j++) {
            if (nums[i - 1] > j) {
                dp[i][j] = dp[i - 1][j];
            } else {
                dp[i][j] = dp[i - 1][j] || dp[i - 1][j - nums[i - 1]];
            }
        }
    }
    return dp[nums.length][mid];
};  
```

### Coin change ###

&emsp;&emsp;完全背包

[leedcode地址](https://leetcode-cn.com/problems/coin-change/)

```
var coinChange = function(coins, amount) {
    var dp = [0];
    for (var i = 1; i <= amount; i++) {
        dp[i] = -1;
        for (var coin of coins) {
            if (coin > i || dp[i - coin] === -1) {
                continue;
            }
            if (dp[i] === -1) {
                dp[i] = dp[i - coin] + 1;
            } else {
                dp[i] = Math.min(dp[i - coin] + 1, dp[i]);
            }
        }
    }
    return dp[amount];
};
```

### Combination sum ###

[leedcode地址](https://leetcode-cn.com/problems/combination-sum-iv/)

&emsp;&emsp;完全背包

```
var combinationSum4 = function(nums, target) {
    var dp = [];
    dp[0] = 1;
    for (var i = 1; i <= target; i++) {
        dp[i] = 0;
        for(var num of nums) {
            if (num > i || dp[i - num] === 0) {
                continue;
            } else {
                dp[i] += dp[i - num];
            }
        }
    }
    return dp[target];
};
```

### Ones and zeroes ###

[leedcode地址](https://leetcode-cn.com/problems/ones-and-zeroes/)

```
var findMaxForm = function(strs, m, n) {
    var dp = [];
    for(var mi = 0; mi <= m; mi++) {
        dp[mi] = [];
        for(var ni = 0; ni <= n; ni++) {
            dp[mi][ni] = [];
            for(var j = 0; j <= strs.length; j++) {
                if (j === 0) {
                    dp[mi][ni][j] = 0;
                    continue;
                }
                var str = strs[j - 1];
                var s0 = 0;
                var s1 = 0;
                for(var index = 0; index < str.length; index++) {
                if (str.charAt(index) === "0") {
                    s0++;
                } else {
                    s1++;
                }
            }
            if (s0 > mi || s1 > ni) {
                dp[mi][ni][j] = dp[mi][ni][j - 1];
            } else {
                dp[mi][ni][j] = Math.max(dp[mi][ni][j - 1], dp[mi - s0][ni - s1][j - 1] + 1);
            }
        }
    }
  }
  return dp[m][n][strs.length];
};
```
