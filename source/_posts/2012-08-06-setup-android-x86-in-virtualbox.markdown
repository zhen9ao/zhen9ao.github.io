---
layout: post
title: "在VirtualBox中配置Android-x86"
date: 2012-08-06 09:38
comments: true
published: true
categories: android
---
Android官方的模拟器速度太慢，性能很差，导致在实际开发中基本只能使用真机进行调试。下面就介绍一下[Android-x86](http://www.android-x86.org/)项目，顾名思义，这个项目是为x86平台设计的Android系统，该系统可以运行在VirtualBox等虚拟机中。效率比官方的模拟器高出很多。

##下载ISO文件

Android-86项目的ISO文件在[这里](http://www.android-x86.org/download)可以下载到。如果想在VirtualBox里面使用，最好下载以`eeepc`结尾的映像文件，兼容性比较好。比如我下载的是：`android-x86-2.3-RC1-eeepc.iso`

##安装

1. 打开VirtualBox，创建虚拟机。
2. `Operating System`：`Linux`，`Version`：`Other Linux`。`内存`：`1024MB`，`硬盘空间`:`512MB`，其他选项可以使用默认配置。
3. 启动虚拟机。选择`Install to Hard Disk`进行安装。安装中要这样选择：
* `Create/Modify partitions`
* `New` -> `Primary` -> `534MB` -> `Bootable` -> `Write` -> `yes` -> `Quit`
4. 分区结束后在列表里就出现了刚创建的分区`sda1`。选择这个分区，格式化成`ext3`格式。接下来，一直选择`yes`.
5. 最后可以创建一个`Fake SD Card`，这样重启过后就安装完成了。
6. 安装完成后要记得将ISO文件移除，这样就可以从安装目录启动了。

##配置启动参数

开发Android就是要适配不同的分辨率，但是默认启动的分辨率一般是不能满足需求的，所以可以配置几个启动参数，配置好后每次启动只需要选择即可。

可以按这个帖子的内容进行配置：--> [StackOverflow](http://stackoverflow.com/a/8273560/401406)

其中可以配置多个分辨率，然后设置成`vga=ask`，这样每次启动的时候就可以根据需要选择不同的分辨率。

##使用

为了能进行adb远程调试，需要将虚拟机的网卡属性修改为`Bridged Adapter`在`Advance`中设置为`PCnet-FAST III(Am79C973)`。

这样启动后虚拟机就和物理机处于同一个网段内，启动虚拟机后按`Cmd` + `Fn` + `F1`进入控制台界面，输入`netcfg`可以查看当前虚拟机分配的IP地址，这样就可以在屋里机中使用`adb connect IP`连接远程机器进行调试了，`Cmd`+`Fn`+`F7`退出控制台模式。

PS：Android-x86对虚拟机的鼠标识别不好，所以要在菜单中设置`Disable Mouse Integration`，这样鼠标就可以在虚拟机中使用了