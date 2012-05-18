---
layout: post
title: "wr703n配置openvpn后记"
date: 2012-05-18 22:44
comments: true
categories: 
---

## 修正

1. iptables的处理
不需要删除`firewall`，不要在`rc.local`中配置iptables，而是在`/etc/firewall.user`中进行配置，配置如下：

		iptables -I FORWARD -o tun0 -j ACCEPT
		iptables -t nat -A POSTROUTING -s 192.168.2.0/24 -j MASQUERADE

2. openvpn自动启动
在`/etc/init.d`目录中创建一个shell启动脚本，仿照其他脚本的格式来写，大致格式如下：

		#!/bin/sh /etc/rc.common
		# /init.d/openvpn
		START=50
		start() {
			/mnt/usb/usr/sbin/openvpn /mnt/usb/blockcn/config.ovpn
		}
		stop() {
			killall openvpn
		}

其中`start`中的路径都要写绝对路径，相对路径不管用的。
然后要在`/etc/rc.d`中创建一个链接，指向刚才创建的脚本，如下：

	ln -s /etc/init.d/openvpn /etc/rc.d/S50openvpn

这样，重启后就搞定了。