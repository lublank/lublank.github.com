---
title: Electron自动更新
author: Blank
date: 2019-10-14 22:00:00
tags: Electron
category: Electron
summary: Electron项目使用electron-builder和electron-updater实现自动更新升级软件，每次发布版本后，客户端即可检测到新版本并提示下载更新，或自动更新。
---

Electron项目使用electron-builder和electron-updater实现自动更新升级软件，每次发布版本后，客户端即可检测到新版本并提示下载更新，或自动更新。

<!--more-->
**优点：**

不需要专用的发布服务器。

1. 代码签名验证不仅适用于macOS，还适用于Windows。
2. 自动生成和发布所有必需的元数据文件和工件。
3. 在所有平台上支持下载进度和分阶段发布。
4. 支持开箱即用的不同提供商（GitHub版本，Amazon S3，DigitalOcean Spaces，Bintray和通用HTTP服务器）。
5. 您只需要2行代码即可使其正常工作。

文档：https://www.electron.build/auto-update

**步骤：**

**1、package.json 配置**

可参考前面的“Electron打包”里面的配置，主要是publish的provider、url选项。

```
"build": {
  "productName": "软件名称",
  "appId": "com.example.desktop",
  "releaseInfo": {
    "releaseNotes": "更新日志： \n1. 实现登录功能 \n2. 实现注册功能",
    "releaseDate": "2020-01-01"
  },
  "publish": [{
    "provider": "generic", // 服务器类型
    "url": "http://test.oss-cn-qingdao.aliyuncs.com/test" // 版本服务器地址
  }],
}
```

注：打包出来会生成latest.yml文件，用于自动更新的配置信息；
latest.yml文件是打包过程生成的文件，为避免自动更新出错，打包后禁止对latest.yml文件做任何修改。
如果文件有误，必须重新打包获取新的latest.yml文件！！！

**2、main.js 主进程配置**

```
import  { autoUpdater } from 'electron-updater';

/**
 * 检查更新
 */
function updateHandle() {
  autoUpdater.setFeedURL({
    provider: 'generic',
    url: "http://test.oss-cn-qingdao.aliyuncs.com/test"
  });
  autoUpdater.on('error', (error) => {
    sendUpdateMessage(`检查更新出错\n${error}`);
  });
  autoUpdater.on('checking-for-update', () => {
    sendUpdateMessage('正在检查更新...')
  });
  autoUpdater.on('update-available', (info) => {
    // sendUpdateMessage(`检查到新版本 ${info.version}`);
    // 打开检查窗口
    createVersionWin(true);
    let releaseMsg = '';
    if (info.releaseNotes) {
      const splitNotes = info.releaseNotes.split(/[^\r]\n/);
      splitNotes.forEach(notes => {
        releaseMsg += `${notes}\n`;
      });
    }
    const data = {
      version: info.version,
      releaseNote: releaseMsg
    };
    // 有新版本，是否下载更新
    targetWin.send('updateAvailable', JSON.stringify(data));
  });
  autoUpdater.on('update-not-available', (info) => {
    if (versionWin && isFirstOpen) {
      isFirstOpen = false;
      versionWin.close();
    }
    sendUpdateMessage(`已经是最新版本${info.version}，无需更新`)
  });
  // 更新下载进度事件
  autoUpdater.on('download-progress', (progressObj) => {
    targetWin.send('downloadProgress', progressObj)
  });
  autoUpdater.on('update-downloaded', (info) => {
    ipcMain.on('isUpdateNow', () => {
      // quit and start update
      autoUpdater.quitAndInstall();
    });
    targetWin.send('isUpdateNow'); // 通知renderer是否立即重启更新
  });

  // 监听渲染线程中用户是否同意下载
  ipcMain.on("isDownload", () => {
    autoUpdater.downloadUpdate();
  });
  // 立即检查
  autoUpdater.checkForUpdates();
}

// 通过main进程发送事件给renderer进程，提示更新信息
function sendUpdateMessage(text) {
  if (targetWin) {
    targetWin.webContents.send('message', text)
  }
}

// 开始检查更新
ipcMain.on("checkForUpdate",(event) => {
  // 执行自动更新检查
  targetWin = event.sender;
  autoUpdater.checkForUpdates();
});
```
在主进程createMainWindow中需要调用一下updateHandle()

**3、渲染页面配置**

在渲染进程renderer中触发自动更新，并添加自动更新事件监听

```
 // 首次打开version窗口时发起检查
ipcRenderer.send("checkForUpdate");
// 监听是否发起检查
ipcRenderer.on('checkForUpdate', (event) => {
  ipcRenderer.send("checkForUpdate"); // 发起检查
});
// 监听是否有消息
ipcRenderer.on('message', (event, text) => {
  this.setState({
    status: 0,
    text,
  });
});
// 检测到有新版本
ipcRenderer.on('updateAvailable', (event, text) => {
  const data = JSON.parse(text);
  this.setState({
    status: 1,
    text: `新版本${data.version}`,
    release: data.releaseNote,
  });
});
// 下载进度
ipcRenderer.on("downloadProgress", (event, progressObj) => {
  const { status } = this.state;
  if (status !== 2 && progressObj.percent < 100) {
    this.setState({
      status: 2,
      downloadPercent: 0,
    });
  }
  this.setState({
    downloadPercent: Math.floor(progressObj.percent) || 0,
  });
});
ipcRenderer.on("isUpdateNow", (event) => {
  this.setState({
    status: 3,
    text: '立即重启应用完成更新？',
  });
});
// 开始下载
startDownload = () => {
  ipcRenderer.send("isDownload");
};
// 立即更新
startUpdate = () => {
  ipcRenderer.send("isUpdateNow");
};

// 页面销毁前移除所有事件监听channel
ipcRenderer.removeAll(["checkForUpdate", "message", "downloadProgress", "isUpdateNow", "updateAvailable"]); // remove只能移除单个事件，单独封装removeAll移除所有事件

```
**4、项目打包**

打包完成后在release文件下生成安装包exe、latest.yml、安装包dmg、zip、latest-mac.yml、json等文件。
Mac包不签名也可以打包成功，但Mac上需要应用签名才能有效自动更新，Windows则不用。

应用软件升级版本，修改package.json中的version属性，例如：改为 version: “1.0.1” (之前为1.0.0)；
再次执行electron-builder打包，Windows下将新版本latest.yml文件和exe文件（MAC下将latest-mac.yml,zip和dmg文件）放到package.json中build -> publish中的url对应的版本服务器地址下；
在应用中触发更新检查，electron-updater自动会通过对应url下的yml文件检查更新。

**5、发布版本到静态服务器**

可以写一个发布脚本：publish.js

```
/**
 * 发布版本到OSS服务器
 * 执行命令：./publish
 */
'use strcit';

const path = require('path');
const fs = require('fs');
const oss = require('ali-oss');
const co = require('co');

const releasePrefix = 'test'; // 发布域，一般使用应用的英文名即可，方便记忆, 也是上方发布配置中的xxxx
const config = {
  accessKeyId: 'xxxxx', // 替换为oss获取到的accessKeyId
  accessKeySecret: 'xxxxx', // 替换为oss获取到的accessKeySecret
  bucket: releasePrefix,
  endpoint: 'oss-cn-qingdao.aliyuncs.com', // 这里设置为OSS的endpoint
  timeout: '60s', // 发布超时时间，根据网络情况调整即可
};

const store = oss(config);
// 会到根目录的release目录拿取已经打包好的应用，可根据实际情况调整
const releaseDir = path.join(__dirname, 'release');

async function publish() {
  // 获取打包的版本信息，可根据实际情况修改代码
  const publishInfo = JSON.parse(fs.readFileSync(path.join(__dirname, 'app', 'package.json')));
  const publishVersion = publishInfo.version;
  // 更新文件
  let releaseFiles = [];

  // mac系统配置，获取 `xxxx-x.x.x.dmg` 和 `xxxx-x.x.x-mac.zip` 打包文件，并且发布 latest-mac.json 和 latest-mac.yml 版本文件
    if (process.platform === 'darwin') {
      releaseFiles = fs
        .readdirSync(releaseDir)
        .filter((name) => {
          if (/-mac.yml$/.test(name)) {
            return true;
          }
          if (/(.dmg|-mac.zip)$/.test(name)) {
            return name.includes(publishVersion)
          }
          return false;
        });
      // 更新所需依赖
      // releaseFiles.push('latest-mac.yml');
    }
  // windows系统配置，获取 `xxxx Setup x.x.x.exe`(NSIS打包) ，并且发布 latest.yml 版本文件
    if (process.platform === 'win32') {
      releaseFiles = fs
        .readdirSync(releaseDir)
        .filter((name) => {
          if (/.yml$/.test(name)) {
            return true;
          }
          if (/(.exe.blockmap|.exe|.nsis.7z)$/.test(name)) {
            return name.includes(publishVersion);
          }
          return false;
        });
      // releaseFiles.push('latest.yml');
    }
  console.log(releaseFiles);
  // 推送文件至oss
  const promises = releaseFiles.map((name) => {
    const fullPath = path.join(releaseDir, name);
    return co.wrap(store.put.bind(store))(`${releasePrefix}/${name}`, fullPath);
  });
  console.log('Publish start...');
  await Promise.all(promises)
    .then(() => console.log('Publish Success!'))
    .catch(error => console.error(error));
}

publish();
```

Mac 执行 `./publish`
Windows 执行 `node .\pulish.js`

