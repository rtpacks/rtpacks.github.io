---
title: 'net: socket、websocket、http'
date: 2024-07-07 01:04:47
categories: 网络
tags: 网络
---

## socket、websocket、http

互联网分 5 层：物理层，链路层，网络层，传输层，应用层。常用的TCP, UDP 属于传输层，HTTP, WEBSOCKET 属于应用层。

如果不想用 HTTP WS 这些协议通信，可以基于 TCP, UDP 创建 socket，服务器和客户端使用自定义协议进行通信，比如消息体里什么字段代表什么意思。HTTP, WS 可以视为已经封装好的 SOCKET 应用协议。

WS 与 HTTP 的关系：WS 在创建连接时使用了 HTTP，连接后的通信就与 HTTP 无关了。
