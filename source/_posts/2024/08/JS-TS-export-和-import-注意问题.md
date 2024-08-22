---
title: 'JS/TS: export 和 import 注意问题'
date: 2024-08-21 09:58:16
categories: JS/TS
tags: JS/TS
---

## export 和 import 注意问题

在默认工具库文件中，一般会有一个默认的 export 数据，例如 `export default function fn() {}` 或者 `export default {}`，这种默认导出在对外统一导出时会出现 `default` 覆盖问题。

```typescript
// useSplitter.ts
export default function useSplitter() {}

// useVisible.ts
export default function useVisible() {}

// index.ts
export * from "./useSplitter"
export * from "./useVisible"
```

useVisible 的 `default export` 会将 useSplitter 的 `default export` 覆盖，只能通过具体的文件来引入。



### issue 参考

- [[Bug\] It cannot import useSplitter directly from @rtpackx/vue because useSplitter is overridden by another default export. · Issue #14 · rtpacks/toolkit (github.com)](https://github.com/rtpacks/toolkit/issues/14)
