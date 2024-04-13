---
title: 'TypeError: opts.node.rangeBy is not a function'
date: 2023-02-27 23:42:25
categories: Stylelint
tags:
- postcss
- stylelint
---

今天遇到 `TypeError: opts.node.rangeBy is not a function` 的问题，网上寻找一番后发现这个解决方案，但是并不适合，因为我的版本已经高于问题版本。

#### postcss 版本过高

```shell
Fix "TypeError: opts.node.rangeBy is not a function" with PostCSS 8.4.4
```

- [Fix "TypeError: opts.node.rangeBy is not a function" with PostCSS 8.4.4 · Issue #5766 · stylelint/stylelint (github.com)](https://github.com/stylelint/stylelint/issues/5766)

#### 删除空的style

根据这个插件往回找，尝试了将空的 `<style lang="less">` 删除，问题解决，这或许是边界判断问题。

```vue
<script setup lang="ts"></script>

<template>
  <div>Share</div>
</template>

// delete
<style lang="less"></style>
```

#### 设置stylelint --custom-syntax postcss

在使用stylelint时可以选择单独设置less，scss，也可以选择直接用postcss

```json
"*.{scss,sass}": "stylelint --fix --custom-syntax postcss-scss",
"*.less": "stylelint --fix --custom-syntax postcss-less",

或者

"*.{scss,sass,less,css}": "stylelint --fix --custom-syntax postcss",
```



#### 其他

- [Fix TypeError: opts.node.rangeBy is not a function_一抹阳光&-CSDN博客](https://blog.csdn.net/qq_45759413/article/details/129261144)

- [TypeError: opts.node.rangeBy is not a function | Az's Blog (azin-cn.github.io)](https://azin-cn.github.io/2023/02/27/TypeError-opts-node-rangeBy-is-not-a-function/)

