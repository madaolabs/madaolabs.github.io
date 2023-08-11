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

1. background(service_worker)
2. action
3. options_ui
4. content_scripts
5. **特别提到 injected_script 不在 manifest 配置中, 但是可以通过 content_scripts 给页面插入 injected_script**

![](manifest.webp)

#### 各部分之间的通信

相关文档: https://developer.chrome.com/docs/extensions/mv3/messaging/

1. content 和 background 之间的通信
2. popup 和 content 之间的通信
3. popup 和 background 之间的通信

content 和 background 之间的通信实例代码：

- 短连接:
  &nbsp;content_scripts 发送信息给 background:

  ```javascript
  const response = await chrome.runtime.sendMessage({ greeting: "hello" });
  ```

  &nbsp;content_scripts 接收信息:

  ```javascript
  chrome.runtime.onMessage.addListener(function (
    request,
    sender,
    sendResponse
  ) {
    console.log(
      sender.tab
        ? "from a content script:" + sender.tab.url
        : "from the extension"
    );
    if (request.greeting === "hello") sendResponse({ farewell: "goodbye" });
  });
  ```

  &nbsp;background 发送信息给 content_scripts:

  ```javascript
  const response = await chrome.tabs.sendMessage(tab.id, { greeting: "hello" });
  ```

  &nbsp;background 接收信息:

  ```javascript
  chrome.runtime.onMessage.addListener(function (
    request,
    sender,
    sendResponse
  ) {
    console.log(
      sender.tab
        ? "from a content script:" + sender.tab.url
        : "from the extension"
    );
    if (request.greeting === "hello") sendResponse({ farewell: "goodbye" });
  });
  ```

- 长连接:
  &nbsp;content_scripts 发起连接，发送消息，监听消息:

  ```javascript
  var port = chrome.runtime.connect({ name: "content-scripts" });
  port.postMessage({ joke: "Knock knock" });
  port.onMessage.addListener(function (msg) {
    if (msg.question === "Who's there?") port.postMessage({ answer: "Madame" });
    else if (msg.question === "Madame who?")
      port.postMessage({ answer: "Madame... Bovary" });
  });
  ```

  &nbsp;background 监听链接，监听消息，发送消息:

  ```javascript
  chrome.runtime.onConnect.addListener(function (port) {
    port.onMessage.addListener(function (msg) {
      if (msg.joke === "Knock knock")
        port.postMessage({ question: "Who's there?" });
      else if (msg.answer === "Madame")
        port.postMessage({ question: "Madame who?" });
      else if (msg.answer === "Madame... Bovary")
        port.postMessage({ question: "I don't get it." });
    });
  });
  ```

popup 和 content 之间的通信实例代码
- 短连接, 和content与background的短链接一下
- 长连接, popup 或者 content 任意一方都可以发起。

  &nbsp;popup发起连接:

  ```javascript
  const [tab] = await chrome.tabs.query({active: true, lastFocusedWindow: true});
  const port = chrome.tabs.connect(tab.id, { name: '' })
  port.postMessage({ joke: "Knock knock" });
  port.onMessage.addListener(function (msg) {
    if (msg.question === "Who's there?") port.postMessage({ answer: "Madame" });
    else if (msg.question === "Madame who?")
      port.postMessage({ answer: "Madame... Bovary" });
  });
  ```
  &nbsp;content发起连接:

  ```javascript
  var port = chrome.runtime.connect({ name: "content-scripts" });
  port.postMessage({ joke: "Knock knock" });
  port.onMessage.addListener(function (msg) {
    if (msg.question === "Who's there?") port.postMessage({ answer: "Madame" });
    else if (msg.question === "Madame who?")
      port.postMessage({ answer: "Madame... Bovary" });
  });
  ```

  &nbsp;popup和content监听链接:

  ```javascript
  chrome.runtime.onConnect.addListener(function (port) {
    port.onMessage.addListener(function (msg) {
      if (msg.joke === "Knock knock")
        port.postMessage({ question: "Who's there?" });
      else if (msg.answer === "Madame")
        port.postMessage({ question: "Madame who?" });
      else if (msg.answer === "Madame... Bovary")
        port.postMessage({ question: "I don't get it." });
    });
  });
  ```

#### 总结
各部分的通信其实就是各部分中对调用 chrome.runtime, chrome.tabs 的方法，实现监听消息与发送消息。原理上和各部分无关，形式上是一个发布订阅模型。


