---
title: Hexo-Theme-Chic添加iconfont失败的临时解决方案
date: 2022-07-26 23:45:06
tags: 
- Chic
- Hexo
- iconfont
---

在Chic的issue中提到了很多自定义icon的解决方案，但是在2022/07/26查看时，有关于icon的解决方案都被删除了。

- 问题：主页中自定义icon图标，在style.styl中@import "..." 加载失败，加载iconfont字体未404，还未找到原因。

- 解决方案：如果不嫌麻烦，直接在themes/Chic/Chic/layout/_partial/head.ejs中加入 `<link rel="stylesheet" href="/fonts/custom/iconfont.css">` 直接加载即可，注意位置，themes/Chic/source/fonts会被编译加载到public目录下。

### Chic目录结构

```shell
├─languages
├─layout
│  ├─_page
│  ├─_partial
│  └─_plugins
├─scripts
└─source
    ├─css
    │  ├─_highlight
    │  ├─_lib
    │  ├─_page
    │  │  └─_post
    │  └─_partial
    ├─fonts
    │  ├─custom
    │  ├─iconfont
    │  └─lanting
    └─js
```

 `source` 文件夹内的所有的内容都会被编译打包到项目根目录下的 `public` 文件夹。

加入 stylesheet 示例

```ejs
<%# css list %>
<% if (theme.stylesheets !== undefined && theme.stylesheets.length > 0) { %>
    <!-- stylesheets list from _config.yml -->
    <% theme.stylesheets.forEach(url => { %>
    <link rel="stylesheet" href="<%- url_for(url) %>">
    <% }); %>
<% } %>

<%# 新增 %>
<link rel="stylesheet" href="/fonts/custom/iconfont.css">
```



