---
title: 前端：记录一次unocss+eslint的使用
date: 2023-05-19 13:05:10
categories: 前端
tags:
  - unocss
  - eslint
---

### 遇到的坑和解决方案

问题

- `import 'virtual:uno.css';` unresolved 报错
- `import/unresolved error`

解决方案

```ts
// .eslintrc.js
rules: {
  'import/no-unresolved': [
    2,
    {
      ignore: ['^virtual:'],
    },
  ],
}
```

或者

```ts
// eslint-disable-next-line import/no-unresolved
import "virtual:uno.css";
```

### 过程

在原子化 css (`Atomic CSS`) 的技术选型中，首先考虑的一定是`tailwindcss`，但截至本文编写日期 (`2023-05-19`) `tailwindcss` 还未提供属性写法。

个人觉得 `windicss` 的属性写法是最为优雅的，所以准备将项目的 `tailwindcss` 转到 `windicss`，但发现 `windicss` 的更新维护越来越少，所以再经过一番调查后，发现了 `unocss`。`unocss` 是由 `antfu` 发起的一项原子化 css 的项目，可以说是 `tailwindcss` 等其他原子化 css 的超集，详细请看官网 [UnoCSS](https://unocss.dev/)

集成出现的 bug 在上面已经说过，我根据官网配置的 `@unocss/eslint-config` 完成配置，但是还是出现了 **`import 'virtual:uno.css';` unresolved 报错** 问题，经过上面解决后成功集成。

- [重新构想原子化 CSS - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/425814828)
- [import 'virtual:uno.css'\] syntax got unresolved problem? · unocss/unocss · Discussion #2448 (github.com)](https://github.com/unocss/unocss/discussions/2448)

### 更多

- [前端：记录一次 unocss + eslint 的使用\_一抹阳光&的博客-CSDN 博客](https://blog.csdn.net/qq_45759413/article/details/130764781)
- [前端：记录一次 unocss + eslint 的使用 - 掘金 (juejin.cn)](https://juejin.cn/post/7234718074983743546)
