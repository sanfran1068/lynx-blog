layout: post
title: "了解浏览器标签页的休眠策略"
date: 2021-09-01 22:30:01
comments: true
categories: 
- Document
tags:
- Browser
- Chrome
- 浏览器
---

标签页的休眠目前越来越多地被应用在了各大浏览器中，这个特性设计主要通过 Unload 标签页使其进入休眠模式，来达到节约电脑资源、内存的目的。与关闭标签页的效果不同，休眠的标签页是将标签页保持在浏览器中，但是状态为未加载的状态，点击标签页即可重新加载页面内容。这个过程可能会有白屏的情况出现，这是浏览器在重绘页面内容。

主流的浏览器中，Google Chrome 浏览器支持 Tab Freezing，Microsoft Edge 浏览器支持 Tab Sleeping，Opera 浏览器支持 Tab Snoozing，Firefox 浏览自支持 Automatic Tab Unloading。另外，大部分基于 Chrome 内核的浏览器支持了实验特性 Tab Groups，也支持了标签页以组维度休眠的特性功能。

<!-- more -->

### Google and Firefox

谷歌浏览器在2019年将 Tab Freeze 作为实验功能引入了 Chrome 浏览器。然而早在2015年谷歌就曾将一个类似的叫做 Tab Discard 的特性从浏览器中移除。目前 Tab Freeze 已经作为原生特性集成在 Chrome 浏览器中。

Tab Discard 是2015年谷歌新增的意向特性，称该特性能够节省内存。当电脑空闲内存较低时，浏览器会自动地将“uninteresting”的 tab 页丢弃。丢弃的页面会以**隐藏起来的 tab 页**、**没有交互过的页面**为准，越早的页面会先被丢弃。页面被丢弃时，页面会**从系统内存中抹去**（Chrome 的任务管理器中看不到该页面），但是其页面状态会保存在硬盘中，浏览器界面上的表现是没有任何改变的。当切换至被丢弃的页面时，浏览器会重载页面。

Tab Freezing 与上面的 Tab Discard 不一样。当一个 tab 页被冻结，其**页面内容仍然会保留在系统内存中**，但是 tab 页面将**不会消耗 CPU** 、**也不会运行任何后台任务**。举例来说，当我们打开一个数据量很大、性能消耗大的页面，过一阵子不与该页面交互，浏览器会自动冻结该页面，并停止页面上的任何脚本运行，直到再次与该页面交互。

**当浏览器实在需要放开一些内存时，会将冻结的页面丢弃。**

Tab Freezing 是在 Chrome 77 版本中引入的新特性，在 79 版本中，Chrome 就将该特性自动集成在浏览器中，浏览器会**自动地**对某些页面进行冻结和丢弃。

通过在浏览器中打开 chrome://discards 页面，能够对所有页面进行重新加载和从内存中丢弃：

![https://intranetproxy.alipay.com/skylark/lark/0/2022/png/12056524/1670204871308-ea866489-4bb6-4ec6-b21b-c351657eed68.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/12056524/1670204871308-ea866489-4bb6-4ec6-b21b-c351657eed68.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

这两种方式能够有效地解决当浏览器消耗过高的浏览器内存。不过尽管当你的系统内存还很充裕，Chrome浏览器仍然会冻结一些长期没有交互的 tab 页来达到 CPU 消耗减少和节电的目的。未来 Microsoft Edge 浏览器也会逐渐引入这些特性。

目前网上无法查询到具体 Chrome 是采取什么策略去实施这两种休眠策略。

### 参考文章

[[1]How Chrome’s “Tab Freezing” Will Save CPU and Battery](https://www.howtogeek.com/444481/how-chromes-tab-freezing-will-save-cpu-and-battery/)

[[2]Chrome tabs intermittent hang or crash with white screen](https://support.google.com/chrome/a/thread/19713332/chrome-tabs-intermittent-hang-or-crash-with-white-screen?hl=en)