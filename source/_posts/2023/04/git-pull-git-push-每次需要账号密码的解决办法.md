---
title: git pull/git push 每次需要账号密码的解决办法
date: 2023-04-21 17:32:48
categories: Git
tags:
---

### 解决方案

如果希望全局使用

1. `git config --global credential.helper store`
2. `git pull / git push`
3. 输入账号密码，下次就可不再输入

如果希望单个项目使用

1. `git config --local credential.helper store`，将 `global` 改为 `local` 即可
2. `git pull / git push`
3. 输入账号密码，下次就可不再输入
