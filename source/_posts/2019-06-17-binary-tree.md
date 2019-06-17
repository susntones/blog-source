---
title: binary-tree
tags:
  - 算法
categories:
  - 九章算法
date: 2019-06-17 19:18:53
---


## Binary Tree ##

&emsp;&emsp;解决二叉树常用的两种方法。

* 遍历法(traverse),缺点是无法多线程。
* 分治法(divide conquer)。merge store 和 quick stort 都使用了分治法。

### Maximum depth of binary tree ###

&emsp;&emsp;最基本的分治模板，分别计算左右子树，然后合并结果。

[leedcode地址](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

```
var maxDepth = function(root) {
    if (root === null) {
        return 0;
    }
    var leftMaxDepth = maxDepth(root.left);
    var rightMaxDepth = maxDepth(root.right);
    return Math.max(leftMaxDepth, rightMaxDepth) + 1;
};
```

### Balanced binary tree ###

[leedcode地址](https://leetcode-cn.com/problems/balanced-binary-tree/)

```
var isBalanced = function(root) {
    var depth = balanceDepth(root);
    return depth === -1 ? false : true;
};

function balanceDepth(root) {
    if (root === null) {
        return 0;
    }
    var leftBalanceDepth = balanceDepth(root.left);
    var rightBalbanceDepth = balanceDepth(root.right);
    if (leftBalanceDepth === -1 || rightBalbanceDepth === -1 || Math.abs(leftBalanceDepth - rightBalbanceDepth) > 1) {
        return -1;
    }
    return Math.max(leftBalanceDepth, rightBalbanceDepth) + 1;
}
```

### Binary tree maximum path sum ###

[leedcode地址](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)

```
var maxPathSum = function(root) {
    var { singleSum, maxSum } = getMathPath(root);
    return maxSum;
};
  
function getMathPath(root) {
    if (root === null) {
        return {
            maxSum: -Infinity,
            singleSum: 0,
        };
    }
    var leftMathPathObj = getMathPath(root.left);
    var rightMathPathObj = getMathPath(root.right);

    var value = root.val;
    var singleSum = Math.max(leftMathPathObj.singleSum + value, rightMathPathObj.singleSum + value, 0);
    var maxSum = Math.max(leftMathPathObj.maxSum, rightMathPathObj.maxSum, leftMathPathObj.singleSum + value + rightMathPathObj.singleSum);  

    return {
      maxSum,
      singleSum,
    }
}
```

### Lowest common ancestor of a binary tree ###

&emsp;&emsp;当root的左右节点查询p, q结果都不为空的时候，说明p和q一个在左节点，一个在右节点，root就是最近公共父节点。

[leedcode地址](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/)

```
var lowestCommonAncestor = function(root, p, q) {
    if (!root || root === p || root === q) {
        return root;
    }
    var left = lowestCommonAncestor(root.left, p, q);
    var right = lowestCommonAncestor(root.right, p, q);
    if (left && right) {
        return root;
    } else if (!left) {
        return right;
    } else {
        return left;
    }
};
```

### Binary tree level order traversal ###

&emsp;&emsp;数的宽度优先搜索是无法使用递归的，需要使用队列。js中，通过Array.shift实现出队。

[leedcode地址](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)

```
var levelOrder = function(root) {
    var result = [];
    var queue = [];
    if (!root) {
        return result;
    }
    queue.push(root);
    while (queue.length > 0) {
        var levelLength = queue.length;
        var levelArray = [];
        for (var i = 0; i < levelLength; i++) {
            var node = queue.shift();
            levelArray.push(node.val);
            if (node.left) {
                queue.push(node.left);
            }
            if (node.right) {
                queue.push(node.right);
            }
        }
        result.push(levelArray);
    }
    return result;
};
```

### Validate binary search tree ###

&emsp;&emsp;bst的中序遍历是升序的。

[leedcode地址](https://leetcode-cn.com/problems/validate-binary-search-tree/)

```
var isValidBST = function(root) {
    var result = validBST(root);
    return result.isBST;
};
  
function validBST(root) {
    if (root === null) {
        return {
            isBST: true,
            min: Infinity,
            max: -Infinity,
        };
    }
    var leftValid = validBST(root.left);
    var rightValid = validBST(root.right);
    var val = root.val;
    if (leftValid.isBST && rightValid.isBST && val > leftValid.max && val < rightValid.min) {
        var min = root.left ? leftValid.min : val;
        var max = root.right ? rightValid.max : val;
        return {
            isBST: true,
            min,
            max,
        };
    } else {
        return {
            isBST: false,
        };
    }
}
```

### Binary search tree iterator ###

&emsp;&emsp;非递归实现递归可以考虑stack。

[leedcode地址](https://leetcode-cn.com/problems/binary-search-tree-iterator/)

```
var BSTIterator = function(root) {
    this.stack = [];
    this.current = root;
};

/**
 * @return the next smallest number
 * @return {number}
 */
BSTIterator.prototype.next = function() {
    while(this.current !== null) {
        this.stack.push(this.current);
        this.current = this.current.left
    }
    this.current = this.stack.pop();
    node = this.current;
    this.current = this.current.right;
    return node.val;
};

/**
 * @return whether we have a next smallest number
 * @return {boolean}
 */
BSTIterator.prototype.hasNext = function() {
    if (this.current !== null || this.stack.length > 0) {
        return true;
    } else {
        return false;
    }
};
```

### Delete nod in a bst ###

[leedcode地址](https://leetcode-cn.com/problems/delete-node-in-a-bst/)

```
var deleteNode = function(root, key) {
    if (root === null) {
        return null;
    }
    if (key < root.val) {
        root.left = deleteNode(root.left, key);
    } else if (key > root.val) {
        root.right = deleteNode(root.right, key);
    } else {
        if (root.left === null && root.right === null) {
            return null
        } else if (root.left === null && root.right !== null) {
            return root.right;
        } else if (root.right === null && root.left !== null) {
            return root.left;
        } else {
            var pre = root.right;
            var current = pre.left;
            if (current === null) {
                pre.left = root.left
                return pre;
            }
            while (current.left !== null) {
                pre = current;
                current = current.left;
            }
            pre.left = current.right;
            current.left = root.left;
            current.right = root.right;
            return current;
        }
    }
    return root;
};
```
