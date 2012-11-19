---
layout: post
title: "OpenWrt脱机下载"
date: 2012-11-19 12:31
comments: true
categories: OpenWrt
---

一般为了实现脱机下载，可能会考虑搭建一个自用的NAS系统，不过这样的一个系统一般要准备一个小机箱之类的PC来搭建。考虑到耗电和成本的因素，加上OpenWrt很好很强大，所以完全没必要为了脱机下载专门配个Atom芯片的PC，一个路由器足以搞定。

### 准备

一个可以刷OpenWrt的路由器（我用的wr703n），一个大U盘（现在U盘都非常便宜，100多就可以买一个32G的），一个Linux虚拟机或者物理机。

## 配置OpenWrt

刷好OpenWrt之后，只需要先把网络、无线先配置好，然后通过wifi连接路由器。

### 让路由器从U盘引导启动

这么做，有几个好处：

1. 使用`opkg`安装的东西就默认在U盘了，需要装什么就可以尽情的装。
2. 如果以后再对路由器修改配置导致路由器变砖或者无法启动，拔掉U盘就可以从路由器的存储启动了，不用重新刷机。

<!--more-->


需要安装几个包： `kmod-usb-storage`、`block-mount`、`kmod-fs-ext4`。

U盘要在Linux系统中先格式化成`ext4`的文件系统。

修改`/etc/config/fstab`：

	config mount
        option device   		/dev/sda1
        option fstype   		ext4
        option options  		rw,sync
        option enabled  		1
        option enabled_fsck 	1
        option is_rootfs 		1
	config swap
        option device   /dev/sda2
        option enabled  1

运行命令，使`fstab`开机自动启动：

	/etc/init.d/fstable enable
	/etc/init.d/fstable start

创建可以被OpenWrt引导的U盘：

	mkdir /tmp/sda1
	mkdir /tmp/root
	mount /dev/sda1 /tmp/sda1
	mount -o bind / /tmp/root
	cp -a /tmp/root/* /tmp/sda1
	
这时重启以后，路由器就从U盘引导启动了！

### 安装配置`transmission`

安装了transmission的bt客户端，

	opkg update && opkg install transmission-web
	
修改transmission的配置`/etc/conifg/transmission`，需要修改的项如下：
	
	option enable 1

	option config_dir '/mnt/usb/bt'	
	option download_dir '/mnt/usb/bt/done'
	
	option rpc_authentication_required true
	option rpc_bind_address '0.0.0.0'
	option rpc_enabled true
	option rpc_password 'yourpassword'
	option rpc_port 9091
	option rpc_username 'user'
	option rpc_whitelist '127.0.0.1,192.168.1.*'
	option rpc_whitelist_enabled false
	
然后启动transmission的自动启动脚本：
	
	/etc/init.d/transmission enable
	/etc/init.d/transmission start
	
这样，通过浏览器访问路由器的`9091`端口就可以访问网页控制界面了，通过这个界面就可以上传种子文件。

### 安装配置`samba`

有了samba服务，就可以实用finder或者资源管理器访问路由器的目录了，不用每次都用scp那么麻烦。

安装：

	opkg install samba36-server
	
要先创建samba用户并设置密码：

	smbpasswd -a root

修改`/etc/config/samba`，在文件后面添加一个共享目录：

	config sambashare
        option 'name'			'root'
        option 'path'			'/'
        option 'read_only'		'no'
        option 'guest_ok'		'no'
        option 'create_mask'	'0700'
        option 'dir_mask'		'0700'
        
修改`/etc/samba/smb.conf.templet`模板文件，需要修改的内容：

	unix charset = utf-8
	#invalid users = root

启用服务：
	
	/etc/init.d/samba enable
	/etc/init.d/samba start	

这样，一个能进行脱机下载的OpenWrt路由器就配置好了！

PS: 如果路由器同时也配置了openvpn服务，最好在下载的时候关闭openvpn，这样也防止vpn账号被警告。