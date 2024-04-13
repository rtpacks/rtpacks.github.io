---
title: TypeScript类型体操：获取数组中元素对象属性的值作为新类型
date: 2023-03-03 20:58:24
categories: TypeScript类型体操
tags:
- TypeScript类型体操
- TypeScript
---

首先先说**获取数组中元素对象属性的值作为新类型**的解决方案

- 使用 `as const` 强调不可变数组
- 使用 `typeof arr` 获取数组类型
- 使用 `[number]` 获取数组元素对象类型，这是关键！`[number]`表示获取数组对象元素类型，数组对象的 `key` 是 `number` 类型
- 最后指定元素属性 `key` 即可 

```tsx
const arr = [
    {
        key: 'update',
    },
    {
        key: 'delete'
    }
]
```

特别的需要指定 `update, delete` 作为某一个类型提示

```tsx
// 'delete' | 'update'
type OperationalKey = (typeof arr)[number]['key']

```

### 详细介绍

遇到一个比较麻烦的问题，个人在项目中使用 `TypeScript + Vue` 的技术栈，当前的需求是给一个 `Action` 定义类型以获得类型提示，而这个 `Action` 内部有一个 `key` 属性，`key` 是一个字符串但是我希望仅限于某几个字符串。

```tsx
const actions = [
    {
        key: 'add',
        ...
    },
    {
        key: 'delete',
    },
    {
        key: 'update'
    }
]

const onAction = (action: {key: string}) => {
    
}
```

很明显，当我们直接使用 `string` 作为 `key` 的类型，我们只能具体看代码有哪些操作符，如果直接在 `action: {key: type}` 中重新写一遍字符串类型势必麻烦且不好维护，所以如何获取 `actions` 数组中元素对象的属性的值作为新类型就是我们要解决的问题。

对于一个对象，我们可以使用 `keyof` 来取出所有的属性并作为新类型返回，但是我们需要的是属性的值作为新类型，所以某一步操作必然是 `['key']`。

反过来看数组，如何获取数组元素类型？在经过一番查找后，发现使用 `[number]` 是符合要求的，`JavaScript` 的 `number` 类型索引一般是用户通过 `push, unshift` 等其他操作加入的数组，这么解释也许不正确，因为JavaScript数组是一个对象，而 `JavaScript ` 的对象的 `key` 是字符串，详细可以看看  [Array - JavaScript | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array#数组下标)，总的来说，通过非负数字访问数组是我们需要的结果，所以`[number]` 没有任何问题。

可以获取数组元素对象类型后，还必须要注意一点，数组必须是不可变的，通常用 `as const` 来修饰，如果数组可变，那么相关元素属性就可变便无法正确获取。

### 更好的解决方案

通常为了避免魔法数字，魔法字符串，也就是很难理解含义的一系列定义，我们会定义一些常量集合来确定，但有些时候觉得麻烦便可使用上述方法来确定类型。

### 其他

- [typescript 如何从数组的项中获取某个值作为类型？ - SegmentFault 思否](https://segmentfault.com/q/1010000040981136)
- [TypeScript 类型体操：获取数组中元素对象属性的值作为新类型 | Az's Blog (azin-cn.github.io)](https://azin-cn.github.io/2023/03/03/TypeScript类型体操：获取数组中元素对象属性的值作为新类型/)
- [TypeScript 类型体操：获取数组中元素对象属性的值作为新类型 | 一抹阳光& - CSDN](https://blog.csdn.net/qq_45759413/article/details/129327751)
