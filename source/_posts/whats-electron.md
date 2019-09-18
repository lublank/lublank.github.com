---
author: Blank
title: 什么是Electron？
date: 2019-09-16 10:00:10
tags: Electron
category: Electron
summary: Electron是一个框架，用于将基于HTML5 / JavaScript的应用程序打包到多个平台（Windows / MacOS / Linux）的独立桌面应用程序中。
---


Electron是一个框架，用于将基于HTML5 / JavaScript的应用程序打包到多个平台（Windows / MacOS / Linux）的独立桌面应用程序中。

<!--more-->

<img alt="Framework" src="https://user-images.githubusercontent.com/16829113/65102791-df225300-d9fe-11e9-9270-32b2f13d3fa1.png" width="800">

Electron核心是拥有两个或多个并发运行的操作系统级进程：main和render进程。

<img alt="Process" src="https://user-images.githubusercontent.com/16829113/65103228-2eb54e80-da00-11e9-81dc-311b7001657e.png" width="800">

在活动监视器里可以看到多少进程与该程序关联，Electron是主进程，一个Electron Helper是GPU进程，另几个是渲染进程。

这些进程每个都彼此同时运行，并且进程的内存和资源是相互隔离的。

**为什么要多进程？**

这个架构决策源自Chromium。Chromium在一个单独的进程中运行每个选项卡（即webContents实例），这样如果一个选项卡遇到致命错误，它就不会关闭整个应用程序。从这个意义上讲，“Chromium 就像操作系统一样构建，使用多个操作系统进程将网站彼此隔离并与浏览器本身隔离。”因此，每个进程“在自己的地址空间中运行，由操作系统调度，并且可以独立失败。当意外地编写了一个无限循环，然后关闭了它运行的选项卡，而不是整个浏览器。这种架构是为了感谢这种弹性。还有安全原因。

**主进程**

主要流程负责创建和管理BrowserWindow实例和各种应用程序事件。它还可以执行诸如注册全局快捷方式，创建本机菜单和对话框，响应自动更新事件等操作。您应用的入口点将指向将在主进程中执行的JavaScript文件。主进程中提供了Electron API的子集（请参见下图），以及所有node.js模块。

**渲染进程**

渲染过程负责运行应用程序的用户界面，也就是一个web页面，它是webContents的一个实例。渲染器中提供了所有DOM API，node.js API和Electron API的子集（请参见下图）。

不要把BrowserWindow与渲染器进程混淆了，在窗口包含webContents实例之前，实际上不会创建渲染进程，单个窗口中可以有一个或多个webContents，因为一个窗口里可以有多个webview，每个webview都有自己的webContents实例和渲染器进程。

<img alt="electron-apis-venn-diagram" src="https://user-images.githubusercontent.com/16829113/65103291-61f7dd80-da00-11e9-97c8-6b90bfbc3f19.png" width="800">

**进程间通信**

Electron使用进程间通信（IPC）在进程之间进行通信。

利用remote模块可以直接使用主进程模块，就像Menu在渲染器中可用一样，不需要手动IPC调用，但幕后真正发生的是你通过同步IPC调用向主进程发出命令。

使用devtron工具可以查看到使用remote远程模块时发生的所有IPC调用，同步IPC调用可能会有一些性能影响。


**CPU密集型任务**

不要在主进程中进行CPU密集型工作，它将锁定所有渲染器进程，可能会阻塞渲染进程的UI。这些任务应该在一个单独的进程中运行，而不是具有UI的现有渲染器或者主进程中。

最简单的方法是使用Electron-remote，它有一个渲染器进程任务池，可以跨多个进程拆分和平衡工作。

例子：

electron-remote-example.js
```
// This works in the either the main or renderer processes.

const { requireTaskPool } = require('electron-remote');
const work = requireTaskPool(require.resolve('./work'));

console.log('start work');

// `work` will get executed concurrently in separate processes

work().then(result => {
  console.log('work done');
  console.log(result);
});

work().then(result => {
  console.log('work done');
  console.log(result);
});

work().then(result => {
  console.log('work done');
  console.log(result);
});
```

work.js
```
const crypto = require('crypto');

// this usually takes a few seconds
function work(limit = 100000) {
  let start = Date.now();
  n = 0;
  while(n < limit) {
    crypto.randomBytes(2048);
    n++;
  }
  return {
    timeElapsed: Date.now() - start,
  };
}

module.exports = work;
```

注：该项目electron-remote已弃用不再维护。


