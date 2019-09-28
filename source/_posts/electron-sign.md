---
title: Electron签名证书
author: Blank
date: 2019-09-28 22:00:00
tags: Electron
category: Electron
summary: Electron项目发布时，一般需要对应用进行签名，防止代码被篡改或下载到山寨软件，还可以有效避免第三方杀毒软件报毒。
---

Electron项目发布时，一般需要对应用进行签名，下面是购买证书简单说明。

<!--more-->

这里选择是DigiCert的证书，从该链接进去下单，会优惠很多。

https://www.digicert.com/friends/sysdev/

**1、下单后，会有客服确认订单信息是否正确，公司是否属实，联系人是否属实。**

**2、确认完成后，会有右键通知授权订单，完成激活证书。**

**3、激活后，会发邮件生成证书的链接，需要使用的浏览器打开（IE、Firefox、Safari）**
<img alt="Generate certificate" src="https://user-images.githubusercontent.com/16829113/65817754-a4f65400-e23d-11e9-9d77-18d042c28ae2.png" width="800">

**4、完成，生成了一个证书文件在本地**
<img alt="Certificate Generated" src="https://user-images.githubusercontent.com/16829113/65817791-f868a200-e23d-11e9-81fa-dece5dadbb51.png"  width="800">

**5、验证证书**

https://www.digicert.com/code-signing/mac-verifying-code-signing-certificate.htm

这里使用Mac来验证证书
<img alt="Certificate Generated" src="https://user-images.githubusercontent.com/16829113/65817911-30241980-e23f-11e9-8fdb-4400ca84eee1.jpg"  width="800">

如果上面有证书不受信任的警告，需要移除它：
<img alt="Certificate Generated" src="https://user-images.githubusercontent.com/16829113/65817987-c1938b80-e23f-11e9-8e11-dff92da611f9.jpg"  width="800">

<img alt="Certificate Generated" src="https://user-images.githubusercontent.com/16829113/65818025-19ca8d80-e240-11e9-9524-5490a943ed13.jpg"  width="800">

<img alt="Certificate Generated" src="https://user-images.githubusercontent.com/16829113/65818094-cf95dc00-e240-11e9-83d6-28caa97bfa9b.jpg"  width="800">

下载文件后，双击安装在keychain，完成验证。
<img alt="Certificate Generated" src="https://user-images.githubusercontent.com/16829113/65818097-d1f83600-e240-11e9-8121-e16aa07ca4ac.jpg"  width="800">

**6、导出证书**

右键导出，输入证书密码（用于保护证书的），得到的.p12即为可用的证书文件。

附：代码签名教程：https://www.digicert.com/code-signing/support/
