---
title: 'TS: 根据 tsdoc 规范自动生成文档'
date: 2024-08-21 14:39:27
categories: tsdoc
tags: tsdoc
---

## 根据 tsdoc 规范自动生成文档

使用 `rollup-plugin-dts` 打包生成一份包含库中所有类型的 `d.ts` 文件，使用 `api-executor` 利用 `d.ts` 文件生成文档模型，使用 `api-document` 利用文档模型生成 `md` 文件，最后使用 `vitepress` 汇总 `md` 文件并美化展示。
