---
layout: post
title: "Arithmatic subarray and subsequence"
date:   2022-03-05 16:00:00 +0800
tags: algorithm
---

# Question description

This post archives several questions of solving arithmatic sequence
given a list.

<!--more-->

# Question 1

## Description

An integer array is called arithmetic if it consists of at least three
elements and if the difference between any two consecutive elements is
the same.

For example, \[1,3,5,7,9\], \[7,7,7,7\], and \[3,-1,-5,-9\] are
arithmetic sequences.

Given an integer array `nums`, return the number of arithmetic subarrays
of nums.

A subarray is a contiguous subsequence of the array.

## Solution --- two loops

As a arithmatic subarray is contiguous, we can use first loop to iterate
all possible pairs of adjacent element in the `nums`. For the second
loop, we aim to extend each pair to downstream if the interval between
the new downstream element and second element in the pair is equal to
the interval between the two elements in the pair. During this process,
we count the number of possible arithmatic subarray.

``` python
def numOfArmSlices(nums):
    if len(nums) <= 1:
  return 0

    res = 0
    for i in range(len(nums)-1):
  itv = nums[i+1] - nums[i]
  for j in range(i+2, len(nums)):
      if nums[j] - nums[j-1] == itv:
      res += 1
      else:
      break
    return(res)
```

## Solution --- one loop

While the two-loop solution is straight forward, it actually use two
loops, which can lead to longer execution time. When the `nums` is
already a arithmatic list, we need to loop until the end of `nums` every
time we consider a possible pair.

Now let\'s look at this question in another prospective. The key to the
definition of arithmatic subarray is the constant interval between
adjacent elements. Therefore, we can construct another list to store the
interval between adjacent elements.

For example, for `nums = [10, 1, 3, 5, 7, 9, 20]`, we can construct a
list to store the interval `itv = [-9, 2, 2, 2, 2,
    11]`. You can notice that the arithmatic subarray can be found by
finding the consecutive constant interval in `itv`.

To make the code structure simpler, we can skip the building of `itv`
and only use a variable to keep track of the interval last seen. In this
way, we can use one loop to get our answer.

``` python
def numOfArmSlices(nums):
    if len(nums) <= 1:
  return 0

    itv = float('inf')          # Last seen interval. Initialized with infinite value.
    res = 0                     # Our answer
    cnt = 1                     # Counter to keep track of the length of longest arithmatic subarray.
    for i in range(1, len(nums)):
  diff = nums[i] - nums[i-1]
  if diff == itv:         # If next interval equal to the interval last seen.
      res += cnt          # Add current counter to answer.
      cnt += 1            # Increase counter by 1.
  else:                   # If next interval does not equal to the interval last seen.
      itv = diff          # Record new interval
      cnt = 1             # Re-initialize counter.

    return(res)
```

Now let\'s look at the counter increment part. Why we want to increase
the counter by 1 everytime? If you consider a arithmatic sequence with a
length of n (n>=3). You can derive the number of possible arithmatic
subarray with this formula:

`f(n) = G(n - 3 + 1) + G(n - 4 + 1) + ... + G(n - n + 1 )`, where `G(x)`
= max(x, 0) and it is used to prevent negative counts.

If we plug in differnt n into this formula, we can get:

1.  For `n=3`, `f(n) = 1`;
2.  For `n=4`, `f(n) = 2 + 1`;
3.  For `n=5`, `f(n) = 3 + 2 + 1`;
4.  For `n=6`, `f(n) = 4 + 3 + 2 + 1`;

So the increament of possible arithmatic subarray is just `n+1` from the
last counter.

Alternatively, we can derive this formula to descript the relationship
of the number of possible arithmatic subarray between two sequences of
length n and n-1:

`h(n) = (n - 3 + 1) + h(n-1)`

So we can also not add the counter to answer whenever we find longer
arithmatic sequence, but to keep track of the longest arithmatic
sequence. And after we find the longest possible arithmatic sequence, we
use `h(n)` to get the number of possible arithmatic subarray
recursively. In this case we need another function to do the recursion
job.

# Question 2

## Question description

You are given an array of n integers, nums, and two arrays of m integers
each, l and r, representing the m range queries, where the ith query is
the range \[l\[i\], r\[i\]\].

## Solution

A straight forward way to get the result is to look at the intervals of
query location to see if they equal to a constant value.

``` python
def armSubArray(nums, l, r):
    res = [True] * len(l)

    for i in range(len(l)):
  arr = sorted(nums[l[i]: r[i]+1])
  itv = arr[1] - arr[0]
  for j in range(2, len(arr)):
      if arr[j] - arr[j-1] != itv:
      res[i] = False

    return(res)
```

# Question 3

## Question description

Given a list of numbers, find out the number of arithmatic subsequence.
For example, for `nums = [1,2,1,2,4,1,5,10]`, a arithmatic subsequence
is `[2, 5, 10]`.

## Solution --- dynamic programming

While certainly we can solve this question with depth-first search to
generate all possible subsequence, it would take longer time.

We can use dynamic programming to remember the intermediate state and
speed up the solving. To make thing simpler, we consider a relaxed
definition of arithmatic subsequence: weak arithmatic subsequence. It is
defined as subsequence that any pair of the elements in the subsequence
have equal interval. For example, `[1,
    3, 2]` is a weak arithmatic subsequence. Also `[1, 2]` is also a
weak arithmatic subsequence.

Consider a 2-D array, take `arr[i][diff]` as the number of week
arithmatic subsequences end at index `i` with an interval of `diff`. We
can get the relationship of subproblems:

`arr[j][diff] = arr[i][diff] + 1`, if nums\[j\] - nums\[i\] == diff and
j \> i.

To understand this, you can think of it as: because the interval between
pair nums\[i\] and nums\[i+1\] is diff, the subsequences ended at
position i+1 has one more weak arithmatic subsequence compared to
position i.

For example, for `nums = [1, 2, 3]`, for subsequence ended at index 1,
we have only one week arithmatic subsequence `[1, 2]` with interval 1.
Now let\'s move forward. For subsequence edede at index 2, we have two
weak arithmatic subsequence: `[1, 2, 3]` and `[2, 3]`. We add a new weak
arithmatic subsequence formed by index 1 and index 2.

Now so far we have gotten the number of weak arithmatic subsequence, but
how can we get the number of arithmatic subsequence? In fact, the number
of arithmatic subsequence is `arr[i][diff]`.

Again, we look at the example of `nums = [1, 2, 3]`. The arithmatic
subsequence is 1, which is the number of weak arithmatic subsequence at
index 1. We can think of it as if we can add a new element to a weak
arithmatic subsequence, it is guaranteed to become a arithmatic
subsequence. At index 2, we can add `3` to the previous weak arithmatic
subsequence `[1, 2]` at index 1. Thus at index 3, we have one arithmatic
subsequence of `[1, 2, 3]` and one weak arithmatic subsequence `[2, 3]`.

Based on this logic, we can write a solution with dynamic programming.

``` python
from collections import defaultdict
def numArmSlice(nums):
    dp = [defaultdict(int) for _ in range(len(nums))]  # Initialize array to store intermediate state. Default value is 0.
    res = 0                                            # Answer
    for i in range(len(nums)):                         # Outer loop to loop through input nums
  for j in range(i):                             # Inner loop to loop through every location before i
      dp[i][nums[i] - nums[j]] += dp[j][nums[i] - nums[j]] + 1  # Update weak arithmatic subsequence number at i
      res += dp[j][nums[i] - nums[j]]  # Count number of weak arithmatic subsequence at index j that can be extended with nums[i] at index i. These weak arithmatic subsequence is guaranteed to be arithmatic subsequence at index i.
    return res
```

Note that since we initialize our `dp` with `defaultdict(int)`, the
default value is 0. Thus, the adding of new weak arithmatic subsequence
to `dp` will not increase our answer, as `dp[j][nums[i] - nums[j]]` is
`0`.
