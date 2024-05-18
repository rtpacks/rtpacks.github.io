---
title: 'canvas: measureText 函数与省略号'
date: 2024-05-18 15:51:48
categories: canvas
tags:
---

.

## canvas: measureText 函数与省略号

利用 `measureText` 可以做 canvas 的省略号功能，代码示例：

```typescript
const drawText = (text: string, maxWidth: number, x: number, y: number) => {
    const rawWidth = ctx.measureText(text).width;
    let textWidth = rawWidth;
    while (text && textWidth > maxWidth) {
        text = text.substring(0, text.length - 1);
        textWidth = ctx.measureText(`${text}...`).width;
    }

    text = rawWidth === textWidth ? text : `${text}...`;
    ctx.fillText(text, x, y);
};
```

