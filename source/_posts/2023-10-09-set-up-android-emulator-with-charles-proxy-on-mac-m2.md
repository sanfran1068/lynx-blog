---
layout: post
title: "在 Mac(M2核心) 上配置 Android Emulator 并使用 Charles 进行抓包"
date: 2023-04-16 10:53:00
comments: true
categories: 
- Translation
tags:
- Frontend
- Test
---

### 材料准备与下载
1. 安装 Android Studio（如安装最新版，则无需安装 JDK）：[https://developer.android.com/studio](https://developer.android.com/studio)
2. 安装 Charles：[https://www.charlesproxy.com/download/latest-release/](https://www.charlesproxy.com/download/latest-release/)
### 配置 Charles

1. 勾选 Proxy => macOS Proxy
2. Proxy => SSL Proxy Setting![截屏2023-09-22 17.33.41.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/12056524/1695375236863-a77140ec-0975-4a4f-8ef7-263373d00d08.png#clientId=u53487919-f5c7-4&from=ui&id=u25e13825&originHeight=1092&originWidth=1320&originalType=binary&ratio=1&rotation=0&showTitle=false&size=259842&status=done&style=none&taskId=uc08ca907-7386-4694-847e-f7bd15a49a6&title=)
3. 安装证书：Help => SSL Proxying => Install Charles Root Certificate
4. 信任证书，打开钥匙串，搜索 Charles，点击信任，选择始终信任![截屏2023-09-22 17.36.02.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/12056524/1695375391085-2f7d2d77-fb9a-44dc-9383-00640be5c92b.png#clientId=u53487919-f5c7-4&from=ui&id=u59377dee&originHeight=1100&originWidth=1250&originalType=binary&ratio=1&rotation=0&showTitle=false&size=401871&status=done&style=none&taskId=u3e2cdf68-a824-4f46-afff-3ceeaf3d8a3&title=)
5. 下载证书备用：Help => SSL Proxying => Save Charles Root Certificate
### 配置 Android Studio

1. 新建项目 New Project => Empty Activity => SDK 选择 API 28 ("Pie"; Android 9.0）=> Finis
2. 下载必要的 Emulator 与 Device 资源
   - Tools => SDK Manager => SDK Tools，勾选 Android SDK Build-Tools 34、Android Emulator、Android SDK Platform-Tools，点击 Apply 进行下载
   - Tools => Device Manager => Create Device => Choose a device definition (recommend Pixel 6) => Select a system image (recommend ARM Images Pie Android 9.0) => Finish
###  启动 Android Emulator 并获取 root 权限

1. 在 Terminal 中查看有哪些可选 avd（Android Virtual Device）：emulator -list-avds
2. 在 Terminal 中切换到 emulator 可执行文件目录下（cd ~/Library/Android/sdk/emulator），并执行启动可写 emulator 的命令：./emulator -avd [avd name] -writable-system
3. 获取 root 权限，在命令行中以此输入两行命令：
   1. adb root
   2. adb remount
4. 虚拟器开启后，将安卓的安装文件拖入虚拟手机屏幕进行安装
### Android Emulator 安装 charles CA 证书
在第一部配置 Charles 中我们保存了一份 Mac 端的根证书，在 Terminal 中使用命令 openssl x509 -subject_hash_old -in  [证书文件名.pem]，查看输出的一个8位字符，之后将证书名重新命名为 [8位字符].0。<br />使用 adb push [证书文件名] /system/etc/security/cacerts/ 命令将证书上传到安卓虚拟机证书目录下。<br />此时，charles 证书安装完成，在安卓虚拟机的 wifi 设置中指定 charles 的 ip 和端口，即可进行抓包和代理。
