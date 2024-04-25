---
title: 'wsl: wsl2无法调用systemctl'
date: 2024-04-24 10:35:19
categories: wsl
tags: wsl
---

wsl2 （debian）默认不支持systemctl，报错信息：

```shell
System has not been booted with systemd as init system (PID 1). Can't operate
```



在读完 [WSL 中的高级设置配置 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/wsl-config) 配置方法后，发现只配置 `wsl.conf`  并不生效，还是以上错误：

```ini
[boot]
systemd=true
```



需要升级最新的wsl：

```shell
// powershell
wsl --update
```



关闭并重启wsl：

```shell
wsl --shutdown

wsl
```



如果 `wsl --update` 卡住，可以按照以下方式进行：

1. 卸载WSL2，关闭虚拟机平台服务，Windows子系统服务，重启，这样才彻底卸载旧版本的WSL2。
2. 去github上找到新版WSL2的安装包，开启上述关闭的两个服务，重启，然后安装；网好就微软商店直接上。



### 参考

- [WSL2开启systemctl命令简单方法，重启WSL2后仍生效 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/352994199)
- [使用 systemd 通过 WSL 管理 Linux 服务 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/systemd)
- [WSL 中的高级设置配置 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/wsl-config)
- [System has not been booted with systemd as init system (PID 1). Can‘t operate.问题解决方法](https://blog.csdn.net/qq_43685040/article/details/112056242?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1-112056242-blog-133523514.235^v43^pc_blog_bottom_relevance_base4&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1-112056242-blog-133523514.235^v43^pc_blog_bottom_relevance_base4)
