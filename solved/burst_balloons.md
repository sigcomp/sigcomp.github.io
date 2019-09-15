---
layout: page
title: Burst Balloons (Leetcode)
---

## Problem Breakdown
You are given a series of balloons that are positioned side-by-side. Each balloon has an integer written on it. When you burst balloon `i` you get `nums[left] * nums[i] * nums[right]` coins, and the two adjacent balloons slide together, filling the gap left by the balloon you just popped. The goal is to figure out what order in which to burst the balloons such that you end with the maximum number of coins possible.

(to make the problem easier to think about, we pad the given set of balloons with balloons with a value of 1, and we can't pop these balloons)

For example, if we are given balloons [3, 1, 5, 8], popping `nums[2]` can be visualized as:

![example](/assets/solution_img/burst_balloons/example.png "example")

## Solution Explanation

This problem can be solved using dynamic programming. The idea is that the balloon bursting problem can be broken into subproblems, and many of these subproblems are repeated across multiple balloon popping "paths". With dynamic programming, the goal is to only have to solve each subproblem once, and then in the future, if we need to solve the subproblem again, we can simply look up the answer in a table rather than have to compute the answer again. The following python code implements one solution to the problem:

```python
class Solution:
    def maxCoins(self, nums: List[int]) -> int:
        nums = [1] + nums + [1]
        dp = [[0]*len(nums) for i in range(len(nums))]

        for gap in range(2,len(nums)):
            for i in range(0,len(nums)-gap):
                j = i+gap
                for k in range(i+1,j):
                    dp[i][j] = max(dp[i][j],nums[i]*nums[k]*nums[j] + dp[i][k] + dp[k][j])
        
        return dp[0][len(nums) - 1]
```

`dp[i][j]` is the maximum number of coins we get after we burst all of the balloons between `i` and `j` (not including `i` and `j`). We start by initializing all values in `dp` to 0.

![balloons](/assets/solution_img/burst_balloons/first.gif "fdsf")

![2d array](/assets/solution_img/burst_balloons/second.gif "sdfd")

The solution here will use a bottom-up dynamic programming approach, meaning we will start with the smallest subproblems and use those to compute answers to larger subproblems. The maximum number of coins from any group relies on knowing the maximum number of coins each subgroup of the group. For instance, to find the maximum number of coins from a group of length 4, you must know the maximum number of coins from the two subgroups of 3 that make up the group of 4. As `dp` is filled in, notice how each value is added only after the values directly to the left and below have been added; this is a visual representation of the dependency just described.

As for the actual mechanism of finding the maximum number of coins for our group of balloons, for each value of `i`, `j`, and `k` we ask the following question: is the maximum number of coins from balloons between `i` and `k` plus `num[i]*nums[k]*nums[j]` plus the maximum number of coins from balloons between `k` and `j` better than what we already have? If so, update our maximum value. If not, shift `k` down the array.

Once we have finished filling out `dp`, we can simply take the value at `dp[0][len(nums) - 1]` as our answer, since it represents "the maximum number of coins you can get by popping balloons between the first and last baloon".
