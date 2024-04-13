---
title: 'JavaScript: 循环和迭代器'
date: 2024-03-27 23:09:19
categories: JavaScript
tags: JavaScript
---

## 循环和迭代器

从用途来看，迭代器跟 for 循环颇为相似，都是去遍历一个集合，但是实际上它们存在不小的差别，其中最主要的差别就是：是否通过索引来访问集合。

在 JavaScript 中，`for` 和 `for..of` 就是 for 循环和迭代器的代表：

```js
const arr = [1, 2, 3];

// for 循环
for (let i = 0; i < arr.length; i++) {
  console.log(`index = ${i}, value = ${arr[i]}`);
}

// 迭代器
for (const value of arr) {
  console.log(`value = ${value}`);
}

// 迭代器，调用特殊的方法，它的key是Symbol.iterator，可以理解成 arr.into_iter() 形式，它将返回一个迭代器
arr[Symbol.iterator]();
```

JavaScript 生成迭代器和可迭代对象非常容易：

- 迭代器是指实现迭代器协议的对象(iterator protocol)，其具有特殊属性 `next` 函数的对象，next 函数返回一个 `{value: any, done: boolean}` 形式的对象。
- 可迭代对象是指实现可迭代协议(iterable protocol)的对象，它具有特殊的属性 `Symbol.iterator` 函数，这个函数返回迭代器，一个对象能否使用迭代器就看是否实现 `Symbol.iterator`。
  迭代协议：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols

```javascript
let index = 0;
const iter = {
  next() {
    if (index <= 1) return { value: index, done: false };
    return { value: null, done: true };
  },
};

iter.next(); // {value: 0, done: false}
iter.next(); // {value: 1, done: false}
iter.next(); // {value: null, done: true}
```

迭代器 next 函数返回值是 {value: any, done: boolean} 形式，所以只需要实现一个函数，循环调用一个对象（迭代器） next 函数，当 next 函数给出终止信号 done = true 时，停止函数流程，就能伪实现可迭代。

```rust
function into_iter(arr: number[]) {
     let index = 0;
     return {
         next: () => {
             if(index < arr.length) return { value: arr[index], done: false };
             return { value: null, done: true };
         }
     }
}

const arr = [1, 2];
const iterator = into_iter(arr);
iterator.next(); // {value: 0, done: false};
iterator.next(); // {value: 1, done: false};
iterator.next(); // {value: null, done: true};
```

**流程再分析**

持续迭代终止时需要一个终止信号，它首先需要一个询问的对象，再者需要固定方式来获取终止信息的流程如循环调用 next 函数，现在询问对象是迭代器，循环调用 next 函数的流程就是迭代，也就是迭代迭代器以获取终止信号。

至此，补上迭代迭代器的流程，整体的可迭代就完成了。

```js
function forOf(arr: number[]) {
  const iterator = into_iter(arr);
  while (iterator.next().done === true) break;
}
```

在上面的模拟流程中，可以将迭代器视为单元算子，将可迭代过程视为整体流程，启动迭代后当单元算子给出终止信号时停止流程。
