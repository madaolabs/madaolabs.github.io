---
title: 浏览器插件
date: 2023-08-10 23:08:23
tags:
---

### 介绍
浏览器插件学习体系大致分为两个部分：
1. manifest.json 中字段含义
2. 各部分之间的通信

#### manifest.json
相关文档：https://developer.chrome.com/docs/extensions/mv3/manifest/
主要组成部分:
1. background
2. action
3. options_ui
4. content_scripts
5. **特别提到 injected_script 不在 manifest 配置中, 但是可以通过 content_scripts 给页面插入 injected_script**

![](manifest.webp)

#### 各部分之间的通信
相关文档: https://developer.chrome.com/docs/extensions/mv3/messaging/

1. popup和background之间的通信
2. background和content之间的通信
3. popup和content之间的通信



![](communication.webp)