---
title: 3. 无重复字符的最长子串 | 字符串搜索 | Map | 哈希
date: 2023-02-19 11:36:48
categories: 算法刷题记录
tags:
- leetcode
- 力扣
- 算法
---

### 题目
[3. 无重复字符的最长子串
](https://leetcode.cn/problems/longest-substring-without-repeating-characters/description/)

### 思路
1. 暴力循环
    定义两个 `for`，第一个 `for` 顺序遍历，第二个 `for` 每次从头开始顺序检查，这样即可拿到最长长度，时间复杂度为 `O(n^2)`。
2. map 结构
    定义一个 `while`，利用一个 `map` 记录遍历到的字符，利用内部 `while` 进行检查是否有重复字符

### 代码
```ts
function lengthOfLongestSubstring(s: string): number {

    let l = 0, r = 0, res = 0, len = s.length;
    let map = new Map<string, number>();

    while (r < len) {
        let char = s[r++], count = (map.get(char) || 0) + 1;
        map.set(char, count);

        while (map.get(char) > 1) {
            const left = s[l++]
            map.set(left, map.get(left) - 1)
        }
        res = Math.max(res, r - l); // 此时的r属于后一位，不需要+1
    }

    return res;

};
```



### 其他

- [3. 无重复字符的最长子串 | 字符串搜索 | Map | 哈希_一抹阳光&的博客-CSDN博客](https://blog.csdn.net/qq_45759413/article/details/129108885)
