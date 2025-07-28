---
title: "MCP: 关于 mcp.tool 装饰器下函数参数默认值问题的记录"
date: 2025-04-29 11:32:21
categories:
tags:
---

今天使用 mcp.tool 装饰器进行开发时，遇到了一个比较棘手的问题。主要涉及到被 @mcp.tool() 装饰的 login 函数及其参数默认值设定的相关情况。（如下所示）：

```python
@mcp.tool()
def login(username: str = None, password: str = ""):
pass

@mcp.tool()
def login(username: str | None = None, password: str = ""):
pass

@mcp.tool()
def login(username: str = "", password: str = ""):
"""登录陆吾服务器"""
pass
```

在使用 mcp dev 运行程序时，发现前两种写法存在明显问题。
第一种写法中，程序似乎对参数类型判断有误，甚至要求必须传入一个字符串，而不是按照预期可以接受默认值。第二种写法同样存在问题，参数无法正常传入。只有第三种写法，在参数传递上表现相对正常。

初步推测，这个问题可能出在 mcp 工具生成的 schema 上。可能是在处理函数参数默认值与类型注解时，schema 的生成逻辑出现了偏差，导致在开发模式下参数传递出现异常。
