---
layout: post
title: "解决Xcode4.3.2打开Organizer中的devices就崩溃的问题"
date: 2012-04-20 09:23
comments: true
categories: 
---

##问题描述
这个问题不知是什么原因导致，但是造成了Xcode可以打开项目，可以打开Organizer看文档等操作，但是切换到Devices标签，就会让整个Xcode崩溃。

##解决
在网上找了很多，试了许多方法，比如：TimeMachine恢复到之前的系统状态、重新安装Xcode、修复磁盘权限等，这些办法都无效。Google了一下发现出现这个问题的人不少，StackOverflow上就有很多人提了这个问题，也有很多解决方案，最后还是[这个](http://stackoverflow.com/a/9729656/401406)办法解决了我的问题。
看来是Xcode4.3.2在Keychain处理上的问题，按苹果[官方文档](http://support.apple.com/kb/TS1544?viewlocale=en_US&locale=en_US)重置了Keychain，就解决了。不过这样做的后果就是开发者证书要重新导入，一些软件要提示重新输入密码。
P.S.：也许可以在Keychain中进行修复而不是重置来解决这个问题，遇到同样问题的同学可以试试。