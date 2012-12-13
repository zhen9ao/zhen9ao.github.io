---
layout: post
title: "在TP-Link上安装OpenWrt并配置OpenVPN"
date: 2012-04-10 10:18
comments: true
published: true
categories: OpenVPN OpenWrt
---
## 路由器的选择

要安装OpenWrt，首先要选择一款适合的路由器。可以在OpenWrt的官网中查看支持的[设备列表](http://wiki.openwrt.org/toh/start)，从中选择可以使用的设备。最好选择带USB口的设备（这样可以接U盘，把需要的软件装到U盘中，比如OpenVPN和Python等组件），一般带USB口的设备都是3G路由器或比较高端的路由器，从节约成本的角度考虑，最好是选择3G路由器，高端路由器当然更好，有更强大的CPU。

我选择了[TP-Link wr703n](http://www.tp-link.cn/pages/product-detail.asp?d=225)，这款路由器小巧，方便携带，但功能不弱。

## 准备工作

这里主要考虑的是wr703n只做爬墙使用，不做PPPoE拨号，所以需要准备的东西如下：

1. 网线
1. U盘（或TF卡+读卡器等） 

<!--more-->

## 刷入固件

* 下载对应的ROM文件，wr703n的文件在[这里](http://downloads.openwrt.org/attitude_adjustment/12.09-rc1/ar71xx/generic/)，这是rc1的代码库。
* ROM分两种，分别以factory和sysupgrade固件，以factory结束的用来从官方固件刷OpenWrt，以sysupgrade结束的固件是用来更新已有的OpenWrt。所以，新买的路由器应该选择对应的factory固件。

### 从官方固件升级
从官方固件升级比较简单，登入TP-Link的Web管理界面，在` 软件升级`中选择本地的`openwrt-ar71xx-generic-tl-wr703n-v1-squashfs-factory.bin`文件，就可以刷入OpenWrt了。

### 从OpenWrt升级
从已刷过的固件升级稍微麻烦一些，步骤主要可以参考OpenWrt的[官方文档](http://wiki.openwrt.org/toh/tp-link/tl-wr703n)，大致如下：

1. 将固件`openwrt-ar71xx-generic-tl-wr703n-v1-squashfs-sysupgrade.bin`上传到OpenWrt的`/tmp`目录中
2. 登入OpenWrt管理界面，执行下面的命令
>mtd write openwrt-ar71xx-generic-tl-wr703n-v1-squashfs-sysupgrade.bin firmware
3. 等待命令行返回后，重启路由器就完成了刷新。 

## 网络配置

默认情况下，刚刷入的OpenWrt是没有开启wifi的，所以只能由网线连结到路由器，刚刷入的路由器是没有配置root密码的，也没开启ssh，只能用telnet登陆路由器

> telnet 192.168.1.1

不需要输入密码即可登陆，如需要配置ssh，只需要修改root密码，ssh启用后，telnet将自动失效

> passwd root

OpenWrt需要配置的文件主要有以下几个：

* `/etc/config/network`  配置网络
* `/etc/config/wireless`  配置无线属性
* `/etc/config/dhcp`  配置dhcp等信息
* `/etc/config/fstab`  配置USB设备自动挂载
* `/etc/config/firewall` 配置防火墙设置

### `/etc/config/network`

配置如下

	config interface 'loopback'
        option ifname 'lo'
        option proto 'static'
        option ipaddr '127.0.0.1'
        option netmask '255.0.0.0'
        
	config interface 'wifi'
        option proto 'static'
        option ipaddr '192.168.2.1'
        option netmask '255.255.255.0'
        
	config interface 'wan'
        option proto 'dhcp'
        option ifname 'eth0'

这个配置中`wan`即为与上级路由器屋里相连的网卡接口，动态分配IP地址；`lan`即为无线wifi，IP地址段设置为192.168.2.0/24。

### `/etc/config/wireless`

	config wifi-device  radio0
        option type     mac80211
        option channel  11
        option macaddr  14:e6:e4:e7:dc:00
        option hwmode   11ng
        option htmode   HT20
        list ht_capab   SHORT-GI-20
        list ht_capab   SHORT-GI-40
        list ht_capab   RX-STBC1
        list ht_capab   DSSS_CCK-40
        # REMOVE THIS LINE TO ENABLE WIFI:
        #option disabled 1
        
	config wifi-iface
        option device   radio0
        option network  wifi
        option mode     ap
        option ssid     yourssid
        option encryption psk2
        option key yourpassword

按配置文件中的提示，将`option disabled 1`注释掉即可启动wifi。将`yourssid`和`yourpassword`替换为需要设置的ssid和wifi密码，这个密码有最少8个字符长度要求。

### `/etc/config/dhcp`

这个文件改动很少，可以配置dnsmasq的`resolvfile`，即DNS。可以新建一个文件`/etc/resolv-gpd.conf`，其中的内容如下(这里用的是成都电信的dns，选择自己地区的dns作为备用dns，这样访问国内网络速度要快些)

	nameserver 8.8.8.8
	nameserver 8.8.4.4
	nameserver 199.91.73.222

然后将`/etc/config/dhcp`中的`option resolvfile`选项修改为`/etc/resolv-gpd.conf`，并且配置`wan`口的dhcp，我的配置如下：

	config dnsmasq
		option domainneeded '1'
        option boguspriv '1'
        option localise_queries '1'
        option rebind_protection '1'
        option rebind_localhost '1'
        option local '/lan/'
        option domain 'lan'
        option expandhosts '1'
        option authoritative '1'
        option readethers '1'
        option leasefile '/tmp/dhcp.leases'
        option resolvfile '/etc/resolv-gpd.conf'
        option nohosts '1'
        option nonegcache '1'
        option strictorder '1'

	config dhcp 'wifi'
		option interface 'wifi'
		option start '100'
		option limit '150'
		option leasetime '12h'
		
		
## 防火墙配置(iptables)

修改`/etc/firewall.user`文件，配置如下：

	iptables -I FORWARD -o tun0 -j ACCEPT
	iptables -t nat -A POSTROUTING -s 192.168.2.0/24 -j MASQUERADE


## 防火墙配置(`/etc/config/firewall`)

把该文件中的`lan`都改为`wifi`即可。


## USB配置

wr703n只有4MB的闪存，装完OpenWrt以后，只剩--1.4Mb--的(现在rom自带了luci，所以刷好后的可用空间只有900多Kb了)可用空间了，这些空间只够装一些必备的工具，OpenVPN等一些软件就需要装载外挂的USB设备中了

### 安装必备软件

1. `kmod-usb-storage`, `block-mount`  用来挂载usb设备
2. `kmod-fs-ext4`  用来启用对文件系统的支持
3. **`kmod-tun`** 虽然会随`openvpn`一起自动安装，但是将kmod-tun自动装在usb中可能会导致启动openvpn的时候提示tun设备无法找到的错误，所以在这里也将它先装到闪存中
4. `ldconfig`  使得安装在USB中的lib能被找到

>	opkg update
>	opkg install [package name]


将`[package name]`替换为包名，可以连续键入多个包名，以空格隔开

### 配置USB设备

可以先找一台Ubuntu机器或者在Mac上先将U盘分区、格式化成FAT格式，但是在Mac中创建分区时最好创建成Windows格式的分区表，如果创建成GUID格式的话，可能会多处一个分区，导致U盘的可用空间分区名会变成`sda2`

在Mac上分区格式化成功后，就可以将U盘接到路由器上，对其进行格式化，使其成为ext4格式

	mkfs.ext4 -j /dev/sda1

等待结束后，就可以将U盘挂载到路由器上了

	mkdir /mnt/usb
	mount /dev/sda1 /mnt/usb

这时候就可以查看到已有设备中已经挂载了`/dev/sda1`

	df -h

成功mount后，可以修改`/etc/config/fstab`，让USB设备在路由器启动时自动挂载


	config mount
		option uuid 033ab6ac-22ef-40c1-8a31-2dd61de49302
		option target /mnt/usb
	  	option device /dev/sda1
	  	option fstype ext4
	  	option options  rw,sync
	  	option enabled  1
	  	option enabled_fsck 0

其中的`uuid`可以由`blkid`命令查到，最后要启动fatab服务

	/etc/init.d/fstab enable
	/etc/init.d/fstab start

这样就完成了USB设备的配置

## 安装OpenVPN

要在usb设备中安装软件需要在`/etc/opkg.conf`中添加`dest usb /mnt/usb`，之后执行一下命令

	opkg -dest usb install openvpn

安装安成后，可以在`/etc/profile`系统路径中加入openvpn的路径

接下来要配置ldconfig，比较简单，创建文件`/etc/ld.so.conf`，在其中写入需要载入的lib文件路径即可，然后键入命令，启用配置

	ldconfig

最后，上传OpenVPN配置文件，启动OpenVPN，就可以实现在路由器上自由爬墙了～

## 后记

* 可以配合使用chnroute，减少openvpn的流量消耗
* 本文大部分内容都是源于这两个博客，感谢这两个博主的文章：[Lei Yang's Geek Life](http://blog.proadm.net/blog/2011/10/25/Building-a-GFWless-AP-on-WR703n/) 和 [Ratazzis's Blog](http://www.ratazzi.org/2012/02/09/install-openvpn-and-openwrt-on-wr703n/)

##PS:

首先将openvpn的默认配置文件复制到`/etc/config`中

	cp /mnt/usb/etc/config/openvpn /etc/config/

修改默认配置文件中的第一个配置

	config openvpn custom_config

        # Set to 1 to enable this instance:
        option enabled 1

        # Include OpenVPN configuration
        option config /mnt/usb/vpn/config.ovpn
        
主要将`enable`设置为`1`，配置文件指向自己的ovpn文件。

然后将启动脚本复制到`/etc/init.d/`中

	cp /mnt/usb/etc/init.d/openvpn /etc/init.d/

修改启动脚本中的openvpn二进制文件位置，改成/mnt/usb/开头的位置。

最后：

	/etc/init.d/openvpn enable
	
这样就配置好了openvpn和自动启动

##PPS:
	
OpenVPN配置文件

首先，使用[chnroute](http://code.google.com/p/chnroutes/wiki/Usage)配置openvpn，让访问国内网络不走vpn的链路。

使用chnroute的android配置文件即可

	python chnroute.py -p android

这样得到了`vpnup.sh`和`vpndown.sh`文件，并且把下载下来的文件中前面的`alias`内容注释掉，因为在openwrt中不需要这些配置，会导致openvpn启动错误。

将这两个文件上传到路由器的目录中，并将文件增加执行权限

	chmod a+x vpnup.sh && chmod a+x vpndown.sh

接下来是openvpn的config文件，大致配置如下：

	remote yourhost port
	client
	dev tun
	proto tcp
	daemon
	resolv-retry infinite
	nobind
	persist-key
	persist-tun
	ca ca文件绝对路径
	cert crt文件绝对路径
	key key文件绝对路径
	ns-cert-type server
	comp-lzo
	route-delay 2
	route-method exe
	verb 3
	script-security 2
	up vpnup.sh文件绝对路径
	down vpndown.sh文件绝对路径
	log /tmp/openvpn.log

这样，重启后就搞定了。

