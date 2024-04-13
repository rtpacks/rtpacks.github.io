---
title: JavaScript：一些需要注意的JavaScript设计失误
date: 2023-08-13 21:37:24
categories: JavaScript
tags: JavaScript
---

拜读 [JavaScript 的设计失误 - Offline (haoqun.blog)](https://haoqun.blog/zh/2016/javascript-design-regrets-cf9619ba) 博客时发现一些自己未听过，但是在实践具有隐患的语言设计失误，在此做笔记记录。

### 1. Stateful RegExps

很难想象正则对象上的函数是有状态的，如 `test` 是一个有状态的函数，在编写代码时遇到需要多次判断或很长的正则，一般是定义独立的正则变量以便使用。

在编写代码时，请注意以下写法。虽然很少，但不是没有。

```js
const reg = /foo/g;
console.log(reg.test("foo foo")); // true
console.log(reg.test("foo foo")); // false

const a = true;
const reg = /foo/g;

if (a) {
  reg.test("foo foo");
  const flag1 = reg.test("foo foo");
  const flag2 = reg.test("foo foo");
  console.log(flag1, flag2);
  if (flag2) {
    console.log("secondary"); // unprinted 未打印
  }
}
```

TODO

### 引用

- [JavaScript 的设计失误 - Offline (haoqun.blog)](https://haoqun.blog/zh/2016/javascript-design-regrets-cf9619ba)
