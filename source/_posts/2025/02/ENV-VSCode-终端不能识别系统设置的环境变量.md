---
title: 'ENV: VSCode 终端不能识别系统设置的环境变量'
date: 2025-02-27 11:25:57
categories:
tags: ENV
---

默认情况好像是 vscode 会自动还原上一次 terminal 的环境变量，在设置中搜索并修改属性：

```json
"terminal.integrated.persistentSessionReviveProcess": "never"
```

重启 VSCode 即可。



### 参考链接

- [vscode终端不能识别系统设置的环境变量？ - 知乎](https://www.zhihu.com/question/266858097)
