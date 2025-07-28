---
title: Antv 使用汇总
date: 2025-06-06 11:38:48
categories:
tags:
---



| 问题                                                         | 参考                                                                                                           | 建议                                                                                                                                           | 关键词                                                              |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| Antv/g6 this.upsert 创建自定义元素，指定元素 id                        | https://github.com/antvis/G6/issues/6865                                                                     | 如果需求是在一个 drawKeyShape 方法中，多次使用了 upsert 方法（创建了额外元素，同时想控制额外元素的样式），建议通过设置当前元素自定义的 states 状态，然后将设置特定的 attributes 传到 drawKeyShape 函数中，这样控制自定义元素状态 | upsert，自定义元素状态                                                   |
| antv 的 chart 实例尽量不用响应式变量存储，g6和g2                           | https://github.com/antvis/G6/issues/6791；https://g6.antv.antgroup.com/manual/getting-started/integration/vue | 不要使用响应式对象！如果外部需要使用 graph 实例，可以使用这种定义方式 `const graphRef = { value: null as null \| G6.Graph };`。这样还可以解决元素难以点击的问题                              | It is illegal to free an event not managed by this EventBoundary |
| 新版 g6  style.transform  只支持 G.TransformArray 不支持 string 类型 |                                                                                                              | `G.TransformArray`                                                                                                                           | transform，translate，rotate，scale                                 |
