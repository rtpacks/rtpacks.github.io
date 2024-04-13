---
title: JavaScript：原始数据类型toString问题
date: 2024-03-24 18:04:33
categories: JavaScript
tags: JavaScript
---







### toString

今天遇到一个有趣的事情，当以 `1.toString` 的方式调用 `toString` 方法时，会遇到一个语法错误：

```shell
Uncaught SyntaxError: Invalid or unexpected token
```

这是因为 JavaScript 解析器会将数字后面的点视为小数点的一部分，而不是属性访问符。因此，它尝试将 `1.` 解析为一个浮点数，接着遇到了 `toString`，这并不符合语法规则，从而导致错误。

为了正确地调用数字字面量的方法，可以使用以下几种方式：

1. 使用两个点 `..`，第一个点被视为小数点，第二个点被解析为属性访问符。这样，解析器能够正确地识别出你是想访问数字的方法。

   ```javascript
   1..toString(); // 正确
   ```

2. 将数字放在括号中，这样JavaScript解析器会将其视为一个表达式，从而可以直接访问其方法。

   ```javascript
   (1).toString(); // 正确
   ```

3. 使用数字对象的构造函数 `Number`。

   ```javascript
   Number(1).toString(); // 正确
   ```

4. 使用 `[]`

   ```javascript
   1['toString']() // 正确
   ```

以上方法都可以避免初始的语法错误，能够正确地调用数字的 `toString` 方法。

### 装箱（Boxing）

为什么看似原始数据类型，也能够调用方法呢？

在 JavaScript 中，原始数据类型（如数字、字符串、布尔值）虽然不是对象，但它们在访问方法或属性时会被临时包装成相应的**对象类型**，这个过程称为装箱（Boxing）。这就是为什么可以在原始数据类型上调用 `.toString` 方法或其他方法的原因。

当执行 `1['toString']` 时，JavaScript 引擎会临时将数字 `1` 转换为 `Number` 对象，然后在这个临时创建的 `Number` 对象上调用 `toString` 方法。完成调用后，这个临时对象就会被丢弃。这个过程是自动发生的，对开发者来说是透明的。

这种行为不仅适用于数字，还适用于其他原始数据类型。例如，字符串会被临时转换为 `String` 对象，布尔值会被转换为 `Boolean` 对象，从而允许在这些原始数据类型上访问方法和属性。
