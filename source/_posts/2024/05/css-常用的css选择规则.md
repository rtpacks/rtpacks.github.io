---
title: 'css: 常用的css选择规则'
date: 2024-05-16 10:10:31
categories: 常用 css 选择规则
tags: css
---

#### 不含有特定子元素的元素

**alias**

- 某个元素（不包含 | 没有）指定后代

**code**

```less
:not(:has( .child )) 
:not(&:has( .child ))
```
