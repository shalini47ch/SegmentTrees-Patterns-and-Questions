# SegmentTrees-Patterns-and-Questions
# Segment Tree

Segment Tree is a data structure that is useful when we need to perform **range queries** on an array and the array can also be updated.

For example, we might have an array:

```text
[2, 5, 1, 7, 3]
```

and queries like:

```text
What is the sum from index 1 to 3?
What is the minimum from index 0 to 4?
```

Doing this by traversing every element for every query can become expensive.

A Segment Tree allows us to answer these queries in **O(log n)** and also supports point updates in **O(log n)**.

---

## Basic Idea

The array is divided into smaller ranges and each node of the tree represents one range.

For example:

```text
                    [0, 4]
                   /      \
               [0, 2]     [3, 4]
              /    \       /   \
           [0,1]   [2]    [3]  [4]
```

What we store at each node depends on the problem.

For a sum query:

```text
node = sum of the range
```

For a minimum query:

```text
node = minimum value in the range
```

The overall structure of the code remains almost the same.

---

# 1. Range Sum Query - Mutable

### LeetCode 307

[LeetCode 307 - Range Sum Query - Mutable](https://leetcode.com/problems/range-sum-query-mutable/)

This is the basic Segment Tree problem.

We need to support two operations:

```text
update(index, value)
```

and

```text
sumRange(left, right)
```

The important thing here is that the array can change, so a normal prefix sum array is not enough.

---

## What does each node store?

Each node stores the **sum of its corresponding range**.

For example:

```text
nums = [1, 3, 5, 7]

             [0,3] = 16
             /      \
        [0,1] = 4   [2,3] = 12
```

So when two child nodes are combined:

```python
def _merge(left, right):
    return left + right
```

---

## Query

While querying a range, there are three possible cases:

### 1. No Overlap

The current segment doesn't have anything to do with the requested range.

```python
if right < start or left > end:
    return 0
```

We return `0` because:

```text
x + 0 = x
```

---

### 2. Complete Overlap

The current segment lies completely inside the requested range.

```python
if left <= start and end <= right:
    return self.tree[node]
```

There is no need to go further down.

---

### 3. Partial Overlap

The current segment partially overlaps the query.

So we query both children:

```python
leftres = self._query(...)
rightres = self._query(...)
```

and combine them:

```python
return self._merge(leftres, rightres)
```

---

## Code

```python
class NumArray:

    def __init__(self, nums: List[int]):
        self.n = len(nums)
        self.tree = [0 for i in range(4 * self.n)]

        self._build(nums, 1, 0, self.n - 1)

    def _merge(self, left, right):
        return left + right

    def _build(self, nums, node, start, end):

        if start == end:
            self.tree[node] = nums[start]
            return

        mid = start + (end - start) // 2

        self._build(nums, 2 * node, start, mid)
        self._build(nums, 2 * node + 1, mid + 1, end)

        self.tree[node] = self._merge(
            self.tree[2 * node],
            self.tree[2 * node + 1]
        )

    def update(self, index: int, val: int) -> None:
        return self._update(
            1,
            0,
            self.n - 1,
            index,
            val
        )

    def _update(self, node, start, end, ind, val):

        if start == end:
            self.tree[node] = val
            return

        mid = start + (end - start) // 2

        if ind <= mid:
            self._update(
                2 * node,
                start,
                mid,
                ind,
                val
            )
        else:
            self._update(
                2 * node + 1,
                mid + 1,
                end,
                ind,
                val
            )

        self.tree[node] = self._merge(
            self.tree[2 * node],
            self.tree[2 * node + 1]
        )

    def sumRange(self, left: int, right: int) -> int:
        return self._query(
            1,
            0,
            self.n - 1,
            left,
            right
        )

    def _query(self, node, start, end, left, right):

        # No overlap
        if right < start or left > end:
            return 0

        # Complete overlap
        if left <= start and end <= right:
            return self.tree[node]

        # Partial overlap
        mid = start + (end - start) // 2

        leftres = self._query(
            2 * node,
            start,
            mid,
            left,
            right
        )

        rightres = self._query(
            2 * node + 1,
            mid + 1,
            end,
            left,
            right
        )

        return self._merge(leftres, rightres)
```

---

# 2. Range Minimum Query

### GeeksforGeeks

[Range Minimum Query - GFG](https://www.geeksforgeeks.org/problems/range-minimum-query/1)

Now we use exactly the same Segment Tree idea, but instead of storing the **sum**, we store the **minimum** value for every range.

This is a good exercise because we don't need to learn a completely new implementation.

We just need to change what the node stores and how two nodes are merged.

---

## What changes?

For Sum:

```python
def _merge(self, left, right):
    return left + right
```

For Minimum:

```python
def _merge(self, left, right):
    return min(left, right)
```

That's the main change.

---

## No Overlap

For the sum problem, we used:

```python
return 0
```

For minimum, we use:

```python
return float('inf')
```

The reason is:

```text
min(x, infinity) = x
```

So an irrelevant segment doesn't affect our answer.

For example:

```text
left result  = 5
right result = infinity
```

Then:

```text
min(5, infinity) = 5
```

which is exactly what we want.

---

## The Query Logic

The same three cases still apply.

### No Overlap

```python
if right < start or left > end:
    return float('inf')
```

### Complete Overlap

```python
if left <= start and end <= right:
    return self.tree[node]
```

### Partial Overlap

```python
leftres = self._query(...)
rightres = self._query(...)

return self._merge(leftres, rightres)
```

So the structure hasn't changed.

---
```python
class Solution:
    #lets use the cncept of segment tree to solve this 
    #now here we need to find minimum in the range
    def _merge(self,left,right):
        return min(left,right)
    
    #now create a helper function to build 
    def _build(self,nums,node,start,end):
        if(start==end):
            self.tree[node]=nums[start]
            return 
        mid=start+(end-start)//2
        self._build(nums,2*node,start,mid)
        self._build(nums,2*node+1,mid+1,end)
        self.tree[node]=self._merge(self.tree[2*node],self.tree[2*node+1])
    
    def _query(self,node,start,end,left,right):
        #consider the case of no overlap
        if(right<start or left>end):
            return sys.maxsize 
        if(left<=start and end<=right):
            return self.tree[node]
        #now find the mid element and perform the overlap cases
        mid=start+(end-start)//2
        leftans=self._query(2*node,start,mid,left,right)
        rightans=self._query(2*node+1,mid+1,end,left,right)
        return self._merge(leftans,rightans)
    
    def rangeMinQuery(self, arr, queries):
        # code here
        # Build tree
        self.n = len(arr)
        self.tree = [0 for i in range(4 * self.n + 1)]
        self._build(arr, 1, 0, self.n - 1)

        # Store answers
        ans = []
        for query in queries:
            left=query[0]
            right=query[1]
            result = self._query(
                1,
                0,
                self.n - 1,
                left,
                right
            )

            ans.append(result)
        return ans
        
        







```

