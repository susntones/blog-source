---
title: linked-list
tags:
  - 算法
categories:
  - 九章算法
date: 2019-06-24 22:21:31
---


# Linked List #

### Remove duplicates from sorted list ###

[leedcode地址](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/)

```
var deleteDuplicates = function(head) {
    if (!head) {
        return null;
    }
    var pre = head;
    var current = head.next;
    var currentVal = head.val;
    while (current) {
        if (current.val === currentVal) {
            pre.next = current.next;
            current = pre.next;
        } else {
            currentVal = current.val;
            pre = current;
            current = current.next;
        }
    }
    return head;
};

```

### Remove duplicates from sorted list II ###

&emsp;&emsp;考虑使用dummy node

[leedcode地址](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/)

```
var deleteDuplicates = function(head) {
    if (!head) {
        return null;
    }
    var dummyNode = new ListNode(null);
    dummyNode.next = head;
    var prePre = dummyNode;
    var pre = head;
    var current = head.next;
    var currentVal = head.val;
    var isDupl = false;
    while (current) {
        if (current.val === currentVal) {
            isDupl = true;
            pre.next = current.next;
            current = pre.next;
        } else {
            currentVal = current.val;
            if (isDupl) {
                prePre.next = current;
            } else {
                prePre = pre;
            }
            isDupl = false;
            pre = current;
            current = current.next;
        }
    }
    if (isDupl) {
        prePre.next = current;
    }
    return dummyNode.next;
};
```

### Reverse linked list ###

[leedcode地址](https://leetcode-cn.com/problems/reverse-linked-list/)

```
var reverseList = function(head) {
    var pre = null;
    while (head) {
        var next = head.next;
        head.next = pre;
        pre = head;
        head = next;
    }
    return pre; // 此处要注意return pre；
};
```

### Reverse linked list II ###

&emsp;&emsp;需要使用dummy node

[leedcode地址](https://leetcode-cn.com/problems/reverse-linked-list-ii/)

```
var reverseBetween = function(head, m, n) {
    var dummyNode = new ListNode(null);
    var preNode = dummyNode;
    var nextNode = null;
    var currentNode = head;
    for (var i = 1; i < m; i++) {
        preNode.next = currentNode;
        preNode = currentNode;
        currentNode = currentNode.next;
    }
    nextNode = currentNode;
    var reversePre = null;
    for (var j = m; j <= n; j++) {
        var next = currentNode.next;
        currentNode.next = reversePre;
        reversePre = currentNode;
        currentNode = next;
    }
    preNode.next = reversePre;
    nextNode.next = currentNode;
    return dummyNode.next;
};
```

### Partition list ###

[leedcode地址](https://leetcode-cn.com/problems/partition-list/)

```
var partition = function(head, x) {
    var leftDummyNode = new ListNode(null);
    var rightDummyNode = new ListNode(null);
    var left = leftDummyNode;
    var right = rightDummyNode;
    while (head) {
        if (head.val < x) {
            left.next = head;
            head = head.next;
            left = left.next;
            left.next = null;
        } else {
            right.next = head;
            head = head.next;
            right = right.next;
            right.next = null;
        }
    }
    left.next = rightDummyNode.next;
    return leftDummyNode.next;
};
```

### Sort list ###

&emsp;&emsp; 归并排序。查找中间元素使用 fast slow point

[leedcode地址](https://leetcode-cn.com/problems/sort-list/)

```
var sortList = function(head) {
    if (head === null || head.next === null) {
        return head;
    }
     var mid = findMid(head);
     var right = mid.next;
     mid.next = null;
     var leftSort = sortList(head);
     var rightSort = sortList(right);
     return mergeList(leftSort, rightSort);
};

function findMid(head) { // 链表查找中间元素常用的方法是“快慢指针”。
    if (!head) {
        return null;
    }
    var slowPoint = head;
    var fastPoint = head.next;
    while (fastPoint && fastPoint.next) {
        slowPoint = slowPoint.next;
        fastPoint = fastPoint.next.next;
    }
    return slowPoint;
}

function mergeList(list1, list2) {
    var dummyNode = new ListNode(null);
    var pre = dummyNode;
    while (list1 && list2) {
        if (list1.val < list2.val) {
            pre.next = list1;
            list1 = list1.next;
        } else {
            pre.next = list2;
            list2 = list2.next;
        }
        pre = pre.next;
    }
    if (list1) {
        pre.next = list1;
    } else {
        pre.next = list2;
    }
    return dummyNode.next;
}
```

### Reorder list ###

&emsp;&emsp;从中间将链表分割，将后半部分链表翻转，之后再合并两个链表。

[leedcode地址](https://leetcode-cn.com/problems/reorder-list/)

```
var reorderList = function(head) {
    if (!head || !head.next) {
        return head;
    }
    var mid = findMid(head);
    var right = reverseList(mid.next);
    mid.next = null;
    var left = head;
    var dummyNode = new ListNode(0);
    var pre = dummyNode;
    while (left) {
        pre.next = left;
        left = left.next;
        pre = pre.next;
        if (right) {
            pre.next = right;
            right = right.next;
            pre = pre.next;
        }
    }
    return dummyNode.next;
};

function findMid(head) {
    if (!head || !head.next) {
        return head;
    }
    var slow = head;
    var fast = head.next;
    while (fast && fast.next) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}

function reverseList(head) {
    var pre = null
    while (head) {
        var next = head.next;
        head.next = pre;
        pre = head;
        head = next;
    }
    return pre;
}
```

### Remove nth node from end of list  ###

&emsp;&emsp;使用fast slow point

[leedcode地址](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

```
var removeNthFromEnd = function(head, n) {
    var dummyNode = new ListNode(0); // 使用dummyNode因为存在删除第一个元素的情况。
    dummyNode.next = head;
    var slow = dummyNode;
    var fast = head;
    for (var i = 0; i < n; i++) {
        fast = fast.next;
    }
    while (fast) {
        slow = slow.next;
        fast = fast.next;
    }
    slow.next = slow.next.next;
    return dummyNode.next;
};
```

### Linked list cycle ###

&emsp;&emsp;使用fast slow point

[leedcode地址](https://leetcode-cn.com/problems/linked-list-cycle/)

```
var hasCycle = function(head) {
    if (!head || !head.next) {
        return false;
    }
    var slow = head;
    var fast = head.next;
    while(fast && fast.next) {
        if (slow === fast) {
            return true;
        }
        slow = slow.next;
        fast = fast.next.next;
    }
    return false;
};
```

### Linked list cycle II ###

&emsp;&emsp;使用fast slow point,假设链表头结点到入环点距离为a，则在快慢指针相遇后，慢指针再走a步到达入环点，这里使用一个临时指针从开始节点走，当临时指针和慢指针相遇的时候，即为入环点。属于特殊技巧，了解即可。

[leedcode地址](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

```
var detectCycle = function(head) {
    if (!head || !head.next) {
        return null;
    }
    var slow = fast = tmp = head;
    while(fast && fast.next) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow === fast) { // 特殊技巧，了解即可。
            while(tmp != slow) {
                tmp = tmp.next;
                slow = slow.next;
            }
            return tmp
        }
    }
    return null;
};
```

### Merge k sorted lists ###

&emsp;&emsp;使用堆进行排序。之后用js实现一个简单堆之后再做。

[leedcode地址](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

### Copy list with random pointer ###

[leedcode地址](https://leetcode-cn.com/problems/copy-list-with-random-pointer/)

### Convert sorted array to binary search tree ###

* 方法一：使用hash表。
* 方法二：一个比较巧妙的方法，不需要额外空间复杂度 1 -> 2 -> 3 -> 4 -> null 转化成 1 -> 1' -> 2 -> 2' -> 3 -> 3' -> 4 -> 4' -> null

[leedcode地址](https://leetcode-cn.com/problems/convert-sorted-array-to-binary-search-tree/)

```
var sortedArrayToBST = function(nums) {
    if (nums === null) {
        return null;
    }
    return buildBst(nums, 0, nums.length - 1);
};

function buildBst(nums, start, end) {
    if (start > end) {
        return null;
    }
    var mid = Math.floor((start + end) / 2);
    var node = new TreeNode(nums[mid]);
    node.left = buildBst(nums, start, mid - 1);
    node.right = buildBst(nums, mid + 1, end);
    return node;
}
```

### Convert sorted list to binary search tree  ###

[leedcode地址](https://leetcode-cn.com/problems/convert-sorted-list-to-binary-search-tree/)

```
var sortedListToBST = function(head) {
    if (head === null) {
        return null
    } else if (head.next === null) {
        return new TreeNode(head.val);
    } else if (head.next.next === null) {
        var node = new TreeNode(head.val);
        node.right = new TreeNode(head.next.val);
        return node;
    }
    var preMidNode = findPreMidNode(head);
    var node = new TreeNode(preMidNode.next.val);
    node.right = sortedListToBST(preMidNode.next.next);
    preMidNode.next = null
    node.left = sortedListToBST(head);
    return node;
};

function findPreMidNode(head) {
    var preMid = null;
    var slow = head;
    var fast = head.next;
    while (fast && fast.next) {
        preMid = slow;
        slow = slow.next;
        fast = fast.next.next;
    }
    return preMid;
}
```