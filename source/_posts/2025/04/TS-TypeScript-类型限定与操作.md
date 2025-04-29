---
title: "TS: TypeScript 类型限定与操作"
date: 2025-04-23 17:54:51
categories:
tags:
---

1. 利用泛型，精确的限定 key 对应的 value 类型

```typescript
interface ExtTrait {
  loading: boolean;
  dropdown: boolean;
  bk_name: string;
  error: boolean;
  edit: boolean;
  editor: InputInstance;
}
type SchemaItem = ExtTrait & TreeNodeLike;
type SchemaItemMap = Record<string, SchemaItem>;
const itemMap = ref<SchemaItemMap>({});
const setItemExtTrait = <T extends keyof ExtTrait>(
  id: Unit,
  trait: T,
  value: ExtTrait[T]
) => {
  itemMap.value[id] = { ...itemMap.value[id], [trait]: value };
};
const getItemExtTrait = <T extends keyof ExtTrait>(
  id: Unit,
  trait: T
): ExtTrait[T] | undefined => {
  return itemMap.value[id]?.[trait];
};

// test
setItemExtTrait("12", "dropdown", true);
```
