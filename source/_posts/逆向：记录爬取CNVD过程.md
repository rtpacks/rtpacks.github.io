---
title: 逆向：记录爬取CNVD过程
date: 2025-05-30 16:02:04
categories:
tags:
---

CNVD 网站具有很强大的反爬机制，它的检测流程如下：

1. 发起请求，第一次返回 script 将设置 cookie，并重定向 pathname 和 search

```html
<script>
    document.cookie = ('_') + ('_') + ('j') + ('s') + ('l') + ('_') + ('c') + ('l') + ('e') + ('a') + ('r') + ('a') + ('n') + ('c') + ('e') + ('_') + ('s') + ('=') + (-~false + '') + (9 - 1 * 2 + '') + ((2) * [2] + '') + (-~[7] + '') + (([2] + 0 >> 2) + '') + (9 + '') + ((1 << 1) + '') + (2 + '') + (1 + 2 + '') + (-~[5] + '') + ('.') + (+!+[] * 2 + '') + ((1 + [2] >> 2) + '') + ((1 | 2) + '') + ('|') + ('-') + (-~false + '') + ('|') + ('k') + ('n') + ('I') + ('E') + ('A') + ('v') + ('t') + ('Y') + ('W') + ('g') + ('W') + ('B') + ('b') + ('t') + ('R') + ('t') + ('l') + ('N') + ('c') + ('y') + ('Q') + ('P') + ([2] * (3) + '') + ('n') + ('X') + ((1 | 2) + '') + ((2) * [2] + '') + ('%') + ((2 ^ 1) + '') + ('D') + (';') + (' ') + ('M') + ('a') + ('x') + ('-') + ('a') + ('g') + ('e') + ('=') + (-~[2] + '') + ((1 + [2]) / [2] + '') + (~~[] + '') + (~~'' + '') + (';') + (' ') + ('P') + ('a') + ('t') + ('h') + ('=') + ('/') + (';') + (' ') + ('S') + ('a') + ('m') + ('e') + ('S') + ('i') + ('t') + ('e') + ('=') + ('N') + ('o') + ('n') + ('e') + (';') + (' ') + ('S') + ('e') + ('c') + ('u') + ('r') + ('e');
    location.href = location.pathname + location.search
</script>
```

2. 利用浏览器的二次请求，携带 cookie 从服务器获取反爬脚本（cookie 加密并选取时间因子生成【动态更新】、**环境监测**），利用脚本生成 cookie，此时的请求才会接入正常的网站服务

只要脚本足够的像人类行为，脚本就检测不出来。为了绕开检测，使用 playwright 应该设置浏览器启动参数：

```ts
browser = await chromium.launch({
  slowMo: 100, // 添加延迟模拟人类操作,单位毫秒
  args: ["--disable-blink-features=AutomationControlled"],
});

const context = await browser.newContext({
  userAgent:
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
  javaScriptEnabled: true,
});
await context.addInitScript(() => {
  Object.defineProperty(navigator, "webdriver", { get: () => false });
});
```
