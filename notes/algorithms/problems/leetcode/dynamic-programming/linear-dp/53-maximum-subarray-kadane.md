# 53. Maximum Subarray

## Problem

Given an integer array `nums`, find the contiguous subarray with the
largest sum.

Example:

``` text
Input:
[-2,1,-3,4,-1,2,1,-5,4]

Output:
6
```

Maximum subarray:

``` text
[4,-1,2,1]
```

------------------------------------------------------------------------

# Solution 1: Brute Force

## Idea

Enumerate every possible subarray.

-   `i`: start index
-   `j`: end index
-   `k`: calculate the sum

Time Complexity:

``` text
O(n^3)
```

Space Complexity:

``` text
O(1)
```

------------------------------------------------------------------------

# Solution 2: Dynamic Programming (Kadane Algorithm)

## Definition

`dp[i]` represents the maximum sum of a subarray ending at index `i`.

## State Transition

``` java
dp[i] = Math.max(nums[i], nums[i] + dp[i - 1]);
```

Meaning:

-   Start a new subarray from `nums[i]`
-   Continue the previous best subarray

## Complexity

Time:

``` text
O(n)
```

Space:

``` text
O(n)
```

------------------------------------------------------------------------

# Space Optimization

Only the previous state is needed.

Final complexity:

``` text
Time: O(n)
Space: O(1)
```

------------------------------------------------------------------------

# Summary

  Approach              Time      Space
  --------------------- --------- -------
  Brute Force           O(n\^3)   O(1)
  Dynamic Programming   O(n)      O(n)
  Kadane Optimization   O(n)      O(1)
