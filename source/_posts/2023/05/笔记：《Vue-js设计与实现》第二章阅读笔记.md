---
title: 笔记：《Vue.js设计与实现》第二章阅读笔记
date: 2023-05-20 00:19:36
categories: 《Vue.js设计与实现》
tags: 笔记
---

### Tree-Shaking

在前端的发展历史中，Tree-Shaking 是由 Rollup.js 普及的，Tree-Shaking 的作用是去除运行时未使用或永远不会执行的代码以减少最终构建产物的体积。

Tree-Shaking 依赖于 ESModule 的静态结构，它的工作流程逻辑概括：未使用的代码不会加入到构建产物中，例如变量、函数。这些未使用的代码有一个名字 `dead code`，指未使用或者已使用但由于上层条件导致永远不会运行的代码。

除了 `dead code`，有一些代码执行了，但对外部没有任何影响也可以进行 Tree-Shaking，比如

```ts
function foo(obj) {
  obj && obj.foo;
}

// main.ts
foo();
```

在这种情况下，由于 JavaScript 是一门动态语言，分析一个函数是否有**副作用**很有难度，比如 obj.foo，看似没有函数执行，但是如果 obj.foo 属性通过 Object.defineProperty 加入了 get 操作捕获或通过 Proxy 代理，就可能在这个捕获操作中发生了副作用，所以 foo 函数会加入到构建产物。

所以 Rollup.js 等打包工具提供了一个方式，使用 `/*#__PURE__*/` 明确这个函数是一个不会发生任何副作用的纯函数。

tips: 在开发的过程中，如果能以纯函数的形式实现，则以纯函数的形式实现。

一般说来，`/*#__PURE__*/` 只用在顶级调用中，可以看到如下代码

```ts
// foo.ts
export default function foo(obj) {
  obj && obj.foo;
}

// main.ts
import foo from "foo.ts";

foo(); // 顶级调用

function bar() {
  foo(); // 函数内调用
}
```

顶级调用是可能发生副作用的，因为执行了 foo 函数。函数内调用也是可能发生副作用的，但是发生的前提是上层条件允许或父函数执行了，否则由于 `dead code` 的存在，这些代码都会被移除。

所以只需要控制顶级调用，就能控制大部分无副作用的函数。

### 构建不同产物

- IIFE
- esm
  - 直接在浏览器中运行 esm-browser
  - 在打包工具中构建 esm-bundler
- cjs

### 更多

- [笔记：《Vue.js 设计与实现》第二章阅读笔记 - 掘金 (juejin.cn)](https://juejin.cn/post/7235091963312947261)
- [笔记：《Vue.js 设计与实现》第二章阅读笔记\_一抹阳光&的博客-CSDN 博客](https://blog.csdn.net/qq_45759413/article/details/130782627)
