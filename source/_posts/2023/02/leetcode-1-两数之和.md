---
title: 1.两数之和 - Hash | 查找元素
date: 2023-01-07 11:50:28
categories: 算法刷题记录
tags:
- leetcode
- 力扣
- 算法
---

### 题目

[1.两数之和](https://leetcode.cn/problems/two-sum/description/)

### 分析

- 暴力 for

- 查找时间为 `O(1)` 的 Hash，包括 Set 和 Map

### 代码

```ts
function twoSum(nums: number[], target: number): number[] {
  const map = new Map<number, number>();
  for(let i=0; i<nums.length; i++) {
    const num = target - nums[i];
    if(map.has(num)) {
      return [map.get(num), i];
    }
    map.set(nums[i], i);
  }
  return [0, 0];
};
```
