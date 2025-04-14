---
title: 'Go: 接口与隐式实现'
date: 2025-03-14 16:02:53
categories: Go
tags: Go
---

类型通过实现一个接口的所有方法来实现该接口。既然无需专门显式声明，也就没有“implements”关键字。

隐式接口从接口的实现中解耦了定义，这样接口的实现可以出现在任何包中，无需提前准备。

因此，也就无需在每一个实现上增加新的接口名称，这样同时也鼓励了明确的接口定义。

### Code

```go
package main

import (
	"fmt"
)

type I interface {
	M()
}

// 类型通过实现一个接口的缩放方法来实现该接口，无需专门的显示声明，也就无需 `implements` 关键字

type T struct {
	S string
}
func (t T) M() {
	fmt.Println(t.S)
}

func main() {
	t := T {"Hi"}
	t.M()
}
```

