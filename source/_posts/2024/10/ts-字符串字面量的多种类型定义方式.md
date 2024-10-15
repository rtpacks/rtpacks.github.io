---
title: 'ts: 字符串字面量的多种类型定义方式'
date: 2024-10-15 17:34:47
categories: TypeScript
tags: TypeScript
---

以往定义字符串字面量时，只会用到固定形式如 `type Color = 'red' | 'blue' | 'green'`，今天无意看到 lib.dom.d.ts 中有些字符串字面量类型的定义非常奇特，可以更精确的表示类型：

```typescript
type AutoFillSection = `section-${string}`;

type AutoFillField = AutoFillNormalField | `${OptionalPrefixToken<AutoFillContactKind>}${AutoFillContactField}`;

type AutoFill = AutoFillBase | `${OptionalPrefixToken<AutoFillSection>}${OptionalPrefixToken<AutoFillAddressKind>}${AutoFillField}${OptionalPostfixToken<AutoFillCredentialField>}`;
```



