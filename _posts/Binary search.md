---
title: Binary search
date: 2017-05-03 17:01:01
categories:
 - IT Technology
tags:
 - Algorithm
toc: false
---

There is a template for binary search, by using which you do not need to think too much about the boundary and how to narrow down the interval / how to calculate mid value.

```c
// check null …
int s = 0, e = array.size() - 1;
while (s + 1 < e) {
    int mid = s + ((e - s) >> 1);
    if (array[mid] == target) { ... }
    if (array[mid] > target) { e = mid; }
    else { s = mid; }
}
// check value
if (array[s] == target) { ... }
else if (array[e] == target) { ... }
```

Compared with normal binary search, the difference is that it cannot find the optimum value at while loop, instead, it sustains two possible value, and then check them after loop.

<!-- more -->

#### Some sample:

1) Find **first** number which is greater than or equals to the target value
http://www.lintcode.com/en/problem/search-insert-position/
``` python
    def searchInsert(self, A, target):
        if A is None or len(A) == 0:
            return 0
        i = 0
        j = len(A) - 1
        while i + 1 < j:
            mid = i + ((j - i) >> 1)
            if A[mid] >= target:
                j = mid
            else:
                i = mid
        if A[i] >= target:
            return i
        elif A[j] >= target:
            return j
        return len(A)
```

2) Find first bad version
Find the **first** value which is bad.
``` python
class Solution:
    """
    @param n: An integers.
    @return: An integer which is the first bad version.
    """
    def findFirstBadVersion(self, n):
        i = 1
        j = n
        while i + 1 < j:
            mid = i + ((j - i) >> 1)
            if SVNRepo.isBadVersion(mid):
                j = mid
            else:
                i = mid
        if SVNRepo.isBadVersion(i):
            return i
        elif SVNRepo.isBadVersion(j):
            return j
        return -1
```

3) Find the interval of a target value
http://www.lintcode.com/en/problem/search-for-a-range/
> (1) find the **first** value that equals to the target
> (2) find the **last** value that equals to the target

``` python
class Solution:
    """
    @param A : a list of integers
    @param target : an integer to be searched
    @return : a list of length 2, [index1, index2]
    """
    def searchRange(self, A, target):
        if A is None or len(A) == 0:
            return [-1, -1]
        i = 0
        j = len(A) - 1
        while i + 1 < j:
            mid = i + ((j - i) >> 1)
            if A[mid] >= target:
                j = mid
            else:
                i = mid
        if A[i] == target:
            left = i
        elif A[j] == target:
            left = j
        else:
            return [-1, -1]
        i = 0
        j = len(A) - 1
        while i + 1 < j:
            mid = i + ((j - i) >> 1)
            if A[mid] <= target:
                i = mid
            else:
                j = mid
        if A[j] == target:
            right = j
        elif A[i] == target:
            right = i
        return [left, right]
```

4) Find Minimum in Rotated Sorted Array
http://www.lintcode.com/en/problem/find-minimum-in-rotated-sorted-array-ii/

``` python
class Solution:
    # @param num: a rotated sorted array
    # @return: the minimum number in the array
    def findMin(self, num):
        i = 0
        j = len(num) - 1
        while i + 1 < j:
            mid = i + ((j - i) >> 1)
            if num[mid] == num[j]:
                j = j - 1
            elif num[mid] > num[j]:
                i = mid
            else:
                j = mid
        if num[i] <= num[j]:
            return num[i]
        return num[j]
```

5) Search in Rotated Sorted Array
http://www.lintcode.com/en/problem/search-in-rotated-sorted-array-ii/

``` python
class Solution:
    """
    @param A : an integer ratated sorted array and duplicates are allowed
    @param target : an integer to be searched
    @return : a boolean
    """
    def search(self, A, target):
        if A is None or len(A) == 0:
            return False
        i = 0
        j = len(A) - 1
        while i + 1 < j:
            mid = i + ((j - i) >> 1)
            if A[mid] == target or A[j] == target:
                return True
            if A[mid] > target and A[mid] < A[j]:
                j = mid
            elif A[mid] < target and A[mid] > A[j]:
                i = mid
            else:
                j = j - 1
                    
        if A[i] == target or A[j] == target:
            return True
        return False
```
