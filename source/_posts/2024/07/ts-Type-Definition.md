---
title: 'ts: Type Definition'
date: 2024-07-22 10:35:21
categories: TypeScript
tags: TypeScript
---

## ts: Type Definition

### Record & keyof any

一直在用 Record，对其中的属性很好奇，今天看了一下定义，发现其中有一个 `keyof any`：

```typescript
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```

查看 `keyof any` 等于的值：

```typescript
type KEY =  keyof any; // string | number | symbol
```

这是因为 JavaScript 所有的数据类型，它的 key 都是由 `string | number | symbol` 这三种组成的。



### env.d.ts

拓展 window 属性：

```typescript
declare interface Window {
  Vue: import("vue").App;
  $vueApp: import("vue").App;
  router: import("vue-router").Router;
}
```



