---
title: "TS: TypeScript 类型操作"
date: 2025-04-23 17:54:51
categories:
tags:
---

1. 利用泛型，精确的限定 key 对应的 value 类型

> keywords: 类型限定，key 对应 value，value 类型

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



2. 利用泛型，在不修改原有代码逻辑上添加新的类型

> keywords: 添加新的类型，不修改原有类型，新类型

```typescript
// 旧类型
export interface IAdvancedQueryField {
  model_name: string;
  model_label: string;
  field_list: Array<{
    field_name: string;
    field_label: string;
    field_type: string;
    elemMatch: 0 | 1;
 
 }>;
}

// 添加新的类型，并且不影响原有类型操作
export interface IAdvancedQuerySimpleField {
  field_name: string;
  field_label: string;
  field_type: string;
  elemMatch: 0 | 1;
}
export interface IAdvancedQueryComplexField {
  model_name: string;
  model_label: string;
  field_list: Array<IAdvancedQuerySimpleField>;
}

export type IAdvancedQueryField<
  T extends IAdvancedQueryComplexField | IAdvancedQuerySimpleField = IAdvancedQueryComplexField,
> = T;


```
