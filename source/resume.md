---
title: Resume
date: 2022-07-26 21:18:00
tags: resume
keywords: 
- resume
- Azin Resume
---

## 个人介绍

- 姓名：魏明锋
- 性别：男
- 生日：2001-06-18
- 邮箱：Mr.MF@foxmail.com

## 工作经验

### 北京知其安科技有限公司    2023.04 - 

前端开发

### 北京知其安科技有限公司    2022.09 - 2023.02

工作内容/成果：

1. 负责封装基础组件，对 ElementUI，Arco Design 二次定制，形成公司特有一套 css 方案，复用程度高
2. 负责抽象表格展示流程，Jira 统计表格业务平均开发时间减少 **32.48%**
   - 抽象 columns，对 columns 操作进行分离，过滤条件、过滤组件二次定制，formatter 根据传入的数据选择展示 map 内容
   - 抽象表格动作，增加 reload、reset 方法，父组件可通过 refs 操作表格触发 request 方法，可根据传出字段 page、limit、size、offset，search 二次处理请求
3. 完成主项目 Vue 版本升至2.7，并编写升级**Wiki**、升级注意事项等，其中为减少未来重复开发，使用 **@vueuse** 库实现兼容 value/modelValue，加深 attrs 与 props 的认识
4. 提出开发环境项目快速迭代发版后浏览器可能出现获取不到最新资源的问题，个人总结两个步骤解决问题，index.html 入口使用协商缓存，Router/PerformanceObserve 处理错误
5. 完成新项目钓鱼邮件 SaaS 的搭建，项目采用 Arco Design UI，其中个人完成邮件配置下四个页面，发现并解决 Mockjs 影响 Blob 对象的 bug

## 项目

- [azin-cn/vue-source-note: the note of vue，源码笔记 (github.com)](https://github.com/azin-cn/vue-source-note)
- [azin-cn/mini-vue: mini-vue的实现 (github.com)](https://github.com/azin-cn/mini-vue)
- [azin-cn/wet-translate: 控制台翻译 - transition -》wet word (github.com)](https://github.com/azin-cn/wet-translate)
- [azin-cn/implement-redux-connect: redux-connect函数的实现](https://github.com/azin-cn/implement-redux-connect)
- [azin-cn/imusic-react: Web-React-Music (github.com)](https://github.com/azin-cn/imusic-react)
- ......

有些未组织好结构，当完成后就尘封在文件夹中，如果有时间会重新整理结构，包括项目的踩坑、常用的配置等。除Notes类型外的项目没有上线运营过，一些项目相似度也很高，所以总结归纳一些不同点即可。

### IMusic
介绍：使用 Vue2 构建 H5 音乐播放器，使用 Vue3+Composition API 重构，使用 React 构建 PC Web 音乐播放器，共三个版本

成果：完整实现 H5 音乐 App，Vue2 版本手写 CSS 提升体验，通过 **STAR法则** 分析项目、组件，**拼音字母导航**，通过 Vuex 联动播放时长等数据，使用节流、事件代理对状态更新频率进行限制，通过自定义指令 v-loading 优化用户体验，**部署上线**

职责：负责页面搭建，组件的设计封装，性能优化等前端工作

技术：Vue，React，Redux，hook，swiper，Git，自定义指令，Webpack，Ant-Design等

难点：歌词滚动与进度条**拖拽**实现，分析并定义组件数据关系，**自定义指令**，控制**并发**加载图片

### TalkToTime

介绍：背景主题为"小程序助力乡村振兴"，通过合作分析，小组成员设定项目为以 "乡村风景分享，乡村建设建议" 为基础的社区交流平台，并使用 Uniapp 重构

成果：完整实现 TalkToTime 原生小程序，包括用户自定义**头像裁剪**，当前区域分享者的**头衔设置**等功能，通过**分包**、构建多个 page 降低用户的加载时间，创新性的使用**无感登录**，仅需打开小程序便可关联用户，优化体验

职责：根据UI构建页面，构想**交互动画**优化用户体验，性能优化如图片懒加载，Git版本管理等

技术：原生小程序，分包，Git，Webpack，Vite，**requestAnimationFrame**等

难点：**组件设计**，页面交互，性能优化，**setData时机**，分析定义组件数据，长列表加载等

### iGreen

介绍：为了提升回收站回收资源效率、结合当前文明生态的目的，设计了 iGreen 小程序。iGreen 具有无感登录、用户、员工、管理员、超级管理等角色功能，通过分包优化用户体验

成果：iGreen 小程序为**简约风格**，但用户需要的功能并不减少，员工的**工作台**功能强大，管理员管理界面较为全面。同时 iGreen 还具有 PC 后台管理页面，目前正以**真实场景测试**

职责：负责小程序前端开发，与后台**同步开发**，封装定义组件，封装工具类

技术：Uniapp，小程序，Vue3，Vuex

难点：**项目设计**，组件设计，数据流向和UI交互，业务流程设计

## 技术

- 深入理解 JavaScript、ES6+、原型，熟悉 CSS，了解浏览器原理

- 熟练 Vue、React，能快速切换不同技术栈
- Node.js + Nest.js 搭建后台服务
- 熟练 TypeScript
- 熟悉 SpringBoot 及数据结构算法、设计模式

## 技能证书

- CET-4，华为 HCIA 认证，蓝桥杯 Java 省赛一等奖，大唐杯省赛二等奖，华为 ICT 大赛省赛三等奖，计算机三级 (网络) 等

## 个人简介

- 热爱技术提升、探究实现原理、分析原因，实现 Mini-Vue、Redux
- 热爱了解开源技术发展，跟踪 Git Issue 分析新技术产生原因和背景，努力参与社区讨论
- 较强沟通协调能力、与同学共同参与微信小程序大赛、探究背景和项目设计
- 较强的分析能力，系统分析项目结构、使用 STAR法则 分析项目、封装组件