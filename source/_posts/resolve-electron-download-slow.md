---
author: Blank
title: 解决Electron安装包首次下载慢的问题
date: 2019-09-19 13:00:00
tags: Electron
category: Electron
summary: 第一次安装依赖环境时都需要下载（类似electron-v3.0.0-darwin-x64.zip）Electron包，网络不代理时可能会下载很慢甚至失败。
---


第一次安装依赖环境时都需要下载（类似electron-v3.0.0-darwin-x64.zip）Electron包，网络不代理时可能会下载很慢甚至失败。

<!--more-->

<img width="800" alt="image" src="https://user-images.githubusercontent.com/16829113/65158882-cbf69e00-da65-11e9-83fb-87bbe2e94eaa.png">


**解决方法：**

可以先下载好所需的相应版本Electron安装包

https://sourceforge.net/projects/electron.mirror/files/

然后将该zip文件和SHASUMS256.txt拷贝到以下目录：


```
~/.electron 
```

Windows下应该是该目录：

```
C:\Users\xxx\.electron
```


如下：

SHASUMS256.txt-3.0.0、electron-v3.0.0-darwin-x64.zip

接着就可以继续安装依赖环境了。