---
title: "V2ray: 记录一次公司VPN和V2rayN结合使用的过程"
date: 2023-05-06 15:52:15
categories: vpn
tags: v2ray, 两个vpn结合使用
---

### 问题

作为 Google, ChatGPT 的重度用户，拥有访问条件是一定的，如果加上再加一层代理，这个时候应该怎么做呢？

在公司使用的 VPN 中，设定了访问 `10.231.*` 的 IP 和 `example.com` 的公司域名，为了避免公司VPN无法访问，需要给 V2ray 加上规则。

个人选用的是绕过大陆，配置直连和端口以及公司的域名

```shell
outboundTag: direct
port: 0-65535
protocol: http tls
inboundTag: socks http socks2 http2

domain:cn.bing.com,
domain:r.bing.com,
domain:th.bing.com,
regexp:.*example\.(com|net)$
regixp:10.231.*
```

### 解决方案

如上
