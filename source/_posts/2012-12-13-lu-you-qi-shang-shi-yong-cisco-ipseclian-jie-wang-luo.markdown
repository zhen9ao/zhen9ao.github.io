---
layout: post
title: "路由器上使用Cisco IPSec连接网络"
date: 2012-12-13 21:30
comments: true
categories: OpenWrt IPSec
---

##安装`vpnc`

连接`IPSec`需要用到`vpnc`这个包，通过`opkg`进行安装。

##配置`vpnc`

在`/etc/vpnc/`目录下创建配置文件，内容如下：

	IPSec gateway server-address
	IPSec ID group-name	
	IPSec secret psk-secret
	Xauth username vpn-username
	Xauth password vpn-password
	NAT Traversal Mode cisco-udp
	
然后运行命令即可启动ipsec了，也创建了一个`tun`的网络设备

	vpn /etc/vpnc/your-config
	
##完善

虽然已经创建了`tun`的设备，但是会发现有些网页可以打开，有些则不行，丢包的情况非常严重，这是因为`vpnc`创建的这个网络设备的`mtu`值是`1412`，而不是一般的`1500`，所以利用`iptables`对数据包进行整形。

在`/etc/firewall.user`中添加如下内容：

	iptables -t mangle -I FORWARD -o tun+ -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
	iptables -t mangle -I FORWARD -i tun+ -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
	iptables -t mangle -A OUTPUT -o tun+ -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
	iptables -t mangle -A POSTROUTING -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
	
重启firewall后网站就能正常打开了。

##路由表

与OpenVPN相同，最好也配置一下路由表，让国内IP不走tun设备，可以依然使用之前的`chnroute`生成的`vpnup.sh`文件，但是要进行一点点修改。把`OLDGW`手动进行设置为上级路由器的网关，如我的上级路由器的网关是`192.168.1.1`，则设置：

	OLDGW=192.168.1.1
	
即可。

启动好vpnc后，运行一下vpnup.sh，就配置好国内路由了。