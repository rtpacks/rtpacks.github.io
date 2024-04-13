---
title: "Bugfix: unplugin-vue-component 找不到 Vant 组件的问题"
date: 2023-05-03 18:14:19
categories: unplugin-vue-component
tags: bugfix
---

### 报错信息

```bash
[plugin:vite:import-analysis] Failed to resolve import "vant/es" from x Does the file exist?

/* unplugin-vue-components disabled */import { Button as __unplugin_components_0 } from 'vant/es'; import 'vant/es/button/style/index'
```

### 现象

如以上报错信息，`vite` 找不到 `Vant` 组件，记录本文时，`vant` `vite` `unplugin-vue-components` 等版本如下

```json
{
  "dependencies": {
    "vant": "^4.2.1"
  },
  "devDependencies": {
    "vite": "^3.2.5",
    "unplugin-vue-components": "^0.24.1"
  }
}
```

`Vant` 官方文档编写的按需引入方式如下：

```ts
import vue from "@vitejs/plugin-vue";
import Components from "unplugin-vue-components/vite";
import { VantResolver } from "unplugin-vue-components/resolvers";

export default {
  plugins: [
    vue(),
    Components({
      resolvers: [VantResolver()],
    }),
  ],
};
```

个人配置如下：

```ts
export default defineConfig({
  plugins: [
    vue(),
    vueJsx(),
    svgLoader({ svgoConfig: {} }),
    Components({
      resolvers: [VantResolver()],
    }),
  ],
  resolve: {
    alias: [
      {
        find: "@",
        replacement: resolve(__dirname, "../src"),
      },
    ],
    extensions: [".ts", ".js"],
  },
  define: {
    "process.env": {},
  },
});
```

### 解决方案

找了挺久的，最后发现是拓展名的问题，不限制拓展名或者加入 `.mjs` 拓展名即可解决。

```ts
export default defineConfig({
-   extensions: [".ts", ".js"],
+   extensions: [".ts", ".js", ".mjs"],
})
```

### 其他

- [Bugfix: unplugin-vue-component 找不到 Vant 组件的问题 - 掘金 (juejin.cn)](https://juejin.cn/post/7228888046870102053)
- [Bugfix: unplugin-vue-component 找不到 Vant 组件的问题_一抹阳光&的博客-CSDN博客](https://blog.csdn.net/qq_45759413/article/details/130475645)
