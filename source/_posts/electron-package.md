---
title: Electron打包
author: Blank
date: 2019-09-23 09:00:00
tags: Electron
category: Electron
summary: 使用electron-builder打包项目，该工具拥有一整套功能：本机应用程序依赖项编译、代码打包压缩、代码签名、支持自动更新、支持多平台安装格式等等。
---

我们可以使用electron-builder打包项目，该工具拥有一整套功能：本机应用程序依赖项编译、代码打包压缩、代码签名、支持自动更新、支持多平台安装格式等等。

<!--more-->

安装

```
yarn add electron-builder --dev
```

然后在package.json中配置参数：

执行命令：
```
"scripts": {
    "build": "concurrently \"yarn build-main\" \"yarn build-renderer\"",
    "build-main": "cross-env NODE_ENV=production webpack --config ./configs/webpack.config.main.prod.babel.js --colors",
    "build-renderer": "cross-env NODE_ENV=production webpack --config ./configs/webpack.config.renderer.prod.babel.js --colors",
    "package": "yarn build && electron-builder build --publish never",
    "package-all": "yarn build && electron-builder build -mwl",
    "package-ci": "yarn postinstall && yarn build && electron-builder --publish always",
    "package-linux": "yarn build && electron-builder build --linux",
    "package-win": "yarn build && electron-builder build --win"
  }
```
配置：
```
"build": {
    "productName": "应用名称：Application",
    "appId": "com.xxx.xxx",
    "releaseInfo": {
      "releaseNotes": "版本更新日志：【新增】xxx",
      "releaseDate": "2019-09-23"
    },
    "publish": [
      {
        "provider": "generic", // 发布类型，静态服务器OSS之类
        "url": "版本发布静态服务器：http://xxx.xxx.xxx/download"
      }
    ],
    "files": [
      "app/dist/",
      "app/app.html",
      "app/main.prod.js",
      "app/includes/" // 列出要打包进asar的文件目录
    ],
    "dmg": {
      "title": "Mac版本的应用名称",
      "background": "安装图标路径：./resources/img/mac-install-bg.png",
      "iconSize": 106,
      "contents": [ // 安装时的图标位置
        {
          "x": 130,
          "y": 170
        },
        {
          "x": 410,
          "y": 170,
          "type": "link",
          "path": "/Applications"
        }
      ],
      "window": { // 安装界面的窗口大小
        "width": 540,
        "height": 380
      }
    },
    "win": { // Windows配置
      "target": [
        {
          "target": "nsis", // 安装类型为NSIS
          "arch": [ // 安装包位数64或32，或32+64
            "x64",
            "ia32"
          ]
        }
      ]
    },
    "nsis": { // 针对上面Windows的NSIS配置
      "installerIcon": "./resources/icon.ico", // 安装图标路径
      "license": "./resources/license_zh.txt", // 服务协议许可证文件的路径
      "language": 30724, // 中文编号，默认为1033（English）
      "oneClick": false, // 一键安装
      "perMachine": true, // 为系统的每个用户安装
      "allowElevation": true, // 提升权限
      "allowToChangeInstallationDirectory": true, // 是否允许用户更改安装目录。
      "createDesktopShortcut": true, // 创建桌面快捷方式
      "runAfterFinish": true, // 安装完成后运行
      "artifactName": "${productName}-${version}.${ext}", // 打包出来的安装包格式
      "uninstallDisplayName": "Application", // 卸载文件的名称
      "deleteAppDataOnUninstall": true, // 卸载后是否删除缓存文件
      "installerSidebar": "./resources/img/install-finish.bmp" // 安装界面的封面
    },
    "nsisWeb": { // Windows的nsisweb的配置
      "installerIcon": "./resources/icon.ico",
      "license": "./resources/license_zh.txt",
      "language": 30724,
      "oneClick": false,
      "perMachine": true,
      "allowElevation": true,
      "allowToChangeInstallationDirectory": true,
      "createDesktopShortcut": true,
      "runAfterFinish": true,
      "artifactName": "${productName}-${version}.${ext}",
      "uninstallDisplayName": "Application",
      "deleteAppDataOnUninstall": true,
      "installerSidebar": "./resources/img/install-finish.bmp"
    },
    "linux": { // 打包linux安装包的配置
      "target": [
        "deb",
        "rpm",
        "snap",
        "AppImage"
      ],
      "category": "Development"
    },
    "directories": {
      "buildResources": "resources", // 打包构建的目录
      "output": "release" // 打包输出文件夹
    }
  }
```

Mac 打包：
```
yarn package
```

Windows打包：
```
yarn package-win
```

补充说明：

使用时上面的注释说明需要删掉。

Windows的NSIS配置：[详细链接](https://www.electron.build/configuration/nsis)

一般来说，在Windows系统打包Windows版本安装包，在MacOS系统打包dmg安装包，因为跨系统打包可能会出现安装时不兼容情况。系统上预先安装好签名证书，打包时electron-builder会自动使用本地的证书进行签名。

**自定义NSIS脚本**

上面的NSIS打包方式为默认的安装过程，也可以采用自定义脚本，可以自定义美化安装卸载过程。比如，安装过程中增加广告轮播图，安装路径、安装进度条，安装完成后提示等。

有两个选项可用：include和script。script允许您提供完全不同的NSIS脚本。

```
"nsis": { // 上面Windows的NSIS配置
  ...
  "include": "默认为build/installer.nsh",
  "script": "默认为build/installer.nsi"
}
```

