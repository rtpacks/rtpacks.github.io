---
title: 前端：window.history修改URL后，跳转再回退不是最后一次的URL
date: 2023-11-18 18:15:43
categories: vue-router
tags: 
- vue-router
- history
---

vue2 (`vue-router3`) 没有去 debug 源码，但是在公司的另外一个项目通过 history (window.history.replaceState) 去修改URL，vue-router3 是实时更新内部的 `currentRoute`，所以在修改参数后，去跳转再回退，是最后一次的URL。

今天 debug `vue-router4` 发现，它的逻辑是记录当前页面最后一次通过 `popstate` 事件变化的 URL，也就是只处理 `popstate` 事件发生时的状态，通过 window.history 的 pushState 和 replaceState 去修改 URL 不会触发 popstate 事件，所以 vue-router4 没有感知是这个原因。

目前来看解决办法：

1. 给当前的路由加上 state，在通过 window.history.replaceState 修改参数后，给当前的路由对象加上 state，返回再把 state 取出来然后回显状态。
2. 不通过 window.history.replaceState 无刷新修改参数，而是通过 vue-router 修改参数，达到更新路由的目的。
3. 既然 vue-router4 通过 popstate 事件处理逻辑，那手动触发 `popstate` 事件就可以了，在通过 window.history 修改参数后，手动 `dispatch popstate` 事件 `window.dispatchEvent(new PopStateEvent("popstate"))` 达到触发 router 的目的，这个和第二种类似。目前副作用未知，我用的这种方式。
4. 手动 hack vue router，让它实时更新，这个我没做尝试。



参考：

1. https://github.com/vuejs/router/issues/366
2. https://github.com/vuejs/router/issues/2035

3. https://github.com/vuejs/router/issues/1270

4. https://github.com/single-spa/single-spa/pull/291
