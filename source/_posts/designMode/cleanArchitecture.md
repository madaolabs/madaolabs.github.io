---
title: 简洁架构
date: 2023-08-21 10:40:27
tags:
---

最近把移动版钱包的架构已经全部应用简洁架构。整体感受这套架构非常适合**重前端**的应用。

### 目录结构

```shell
├src
| ├─_api
| ├─_app
| ├─_domain
| ├─_lib
| ├─_services
| ├─_test
| ├─_ui

```

**\_ui**: 所有的页面，组件，资源 都会在这个目录下。
**\_api**: UI 层去调用这一层，相当于简洁架构的 adapter，负责逻辑的**组装**。
**\_app**: api 层去调用这一层，这里所有的方法要求是纯函数，要求必须走单元测试。负责真正的逻辑处理。
**\_domain**: 核心层，主要定义数据结构和各个数据结构之间的转换。
**\_test**: \_domain 和 \_app 的单元测试。
**\_services**: 依赖的外部服务，比如网络请求，合约，浏览器 API 等。
**\_lib**: 一些公共的配置，公共常量等信息

### migration

大多数应用的都会用到状态管理库, 调用流程通常是: UI 层调用 StateManagement 层.
在 Clean Architecture 中，我将 StateManagement 转换为 \_api。
