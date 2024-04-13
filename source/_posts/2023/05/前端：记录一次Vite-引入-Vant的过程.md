---
title: 前端：记录一次 Vite 引入 Vant 的过程，生产环境不渲染 Vant 组件
date: 2023-05-05 21:25:40
categories: vite
tags: vite, vant
---

### 问题

经过前一个步骤引入 Vant 后（[Vant 组件的问题 | Az's Blog (azin-cn.github.io)](https://azin-cn.github.io/2023/05/03/Bugfix-unplugin-vue-component-找不到-Vant-组件的问题/)），在开发环境下使用是正常的，但是当我在 `production` 环境打包时，又出现了一个新的问题，`vue` 并没有渲染 `vant` 组件标签，而是以 `van-tabs` 的形式出现。

### 解决方案

经过一番排查了对比 `Arco Design` 的引入方式发现，在 `Arco Design` 是将所有的组件导出为一个 `Vue` 插件，最后 `app.use(ArcoVue)` 实现全局注册组件，通过自定义 `unplugin-vue-component` `Resolver`的形式按需引入组件。而在 `Vant` 的官方文档中，我并未找到有关于系统的配置，文档只是简写了 `app.use(Vant)` 或使用 `unplugin-vue-component` 的 `VantResolver()` 的形式 。

- [快速上手 - Vant 4 (gitee.io)](https://vant-contrib.gitee.io/vant/#/zh-CN/quickstart)

最终不得已，我按照 `Arco Design` 的形式在 `main.ts` 中引入 `Vant`，我不知道我的做法正不正确，但是这着实解决了我目前的问题。希望有同学可以回答。

```ts
// main.ts
import Vant from "vant";

app.use(Vant);
```

在 `vite.config.ts` 中

```ts
import Components from "unplugin-vue-components/vite";
import { VantResolver } from "unplugin-vue-components/resolvers";

export default defineConfig({
  plugins: [
    vue(),
    vueJsx(),
    svgLoader({ svgoConfig: {} }),
    configArcoStyleImportPlugin(),
    Components({
      resolvers: [VantResolver()],
    }),
  ],
});
```

### 其他

- [前端：记录一次 Vite 引入 Vant 的过程，生产环境不渲染 Vant 组件 - 掘金 (juejin.cn)](https://juejin.cn/post/7229660119658709049/)
- [前端：记录一次 Vite 引入 Vant 的过程，生产环境不渲染 Vant 组件_一抹阳光&的博客-CSDN博客](https://blog.csdn.net/qq_45759413/article/details/130515966)
