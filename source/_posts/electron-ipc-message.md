---
author: Blank
title: Electron进程间通信
date: 2019-09-21 12:00:00
tags: Electron
category: Electron
summary: Electron进程间通信，可以利用 ipcMain 和 ipcRenderer 模块。
---

Electron进程间通信，可以利用 ipcMain 和 ipcRenderer 模块。

<!--more-->

**主进程与渲染进程间通信**

渲染进程向主进程发送消息：
```
// In main process.
const {ipcMain} = require('electron')
ipcMain.on('asynchronous-message', (event, arg) => {
    console.log(arg) // prints "ping"
    event.sender.send('asynchronous-reply', 'pong')
})

ipcMain.on('synchronous-message', (event, arg) => {
    console.log(arg) // prints "ping"
    event.returnValue = 'pong'
})
```
主进程可以使用同步方法或者异步方法返回信息

```
// In renderer process (web page).
const {ipcRenderer} = require('electron')
console.log(ipcRenderer.sendSync('synchronous-message', 'ping')) // prints "pong"

ipcRenderer.on('asynchronous-reply', (event, arg) => {
    console.log(arg) // prints "pong"
})
ipcRenderer.send('asynchronous-message', 'ping')
```

主进程也可以主动向渲染进程发送消息：

```
// In the main process.
const {app, BrowserWindow} = require('electron')
let win = null

app.on('ready', () => {
    win = new BrowserWindow({width: 800, height: 600})
    win.loadURL(`file://${__dirname}/index.html`)
    win.webContents.on('did-finish-load', () => {
        win.webContents.send('ping', 'whoooooooh!')
    })
})
```

```
<!-- index.html -->
<html>
<body>
  <script>
    require('electron').ipcRenderer.on('ping', (event, message) => {
      console.log(message) // Prints 'whoooooooh!'
      // event.sender.send('asynchronous-messag', 'ping')
    })
  </script>
</body>
</html>
```

渲染进程的监听事件回调函数中，也可以通过 event.sender 来向主进程发送消息。

**渲染进程web页面与webview通信**
  
https://electronjs.org/docs/api/webview-tag#event-ipc-message

```
// 在渲染web页面
const webview = document.querySelector('webview')
// 或 const webview = document.getElementsByTagName('webview'); webview[0].send()...
webview.addEventListener('ipc-message', (event) => {
    console.log(event.channel) // 打印 "pong"
})
webview.send('ping', 'args') // 发送信息
```

```
// 在webview访客页接收信息，并发送信息
const { ipcRenderer } = require('electron')
ipcRenderer.on('ping', (event, arg) => {
    console.log(arg) // 打印args
    ipcRenderer.sendToHost('pong', 'arg1')
})
```

