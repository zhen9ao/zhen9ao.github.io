---
layout: post
title: "wr703n配置openvpn后记"
date: 2012-05-18 22:44
comments: true
published: false
categories: OpenVPN wr703n
---

## 修正

1. iptables的处理
不需要删除`firewall`，不要在`rc.local`中配置iptables，而是在`/etc/firewall.user`中进行配置，配置如下：

{% codeblock %}
iptables -I FORWARD -o tun0 -j ACCEPT
iptables -t nat -A POSTROUTING -s 192.168.2.0/24 -j MASQUERADE
{% endcodeblock %}

<!--more-->

2. openvpn自动启动
在`/etc/init.d`目录中创建一个shell启动脚本，仿照其他脚本的格式来写，大致格式如下：

{% codeblock %}
#!/bin/sh /etc/rc.common
# /init.d/openvpn
START=50
start() {
	/mnt/usb/usr/sbin/openvpn /mnt/usb/blockcn/config.ovpn
}
stop() {
	killall openvpn
}
{% endcodeblock %}

其中`start`中的路径都要写绝对路径，相对路径不管用的。
然后要在`/etc/rc.d`中创建一个链接，指向刚才创建的脚本，如下：

	ln -s /etc/init.d/openvpn /etc/rc.d/S50openvpn
	
3. OpenVPN配置文件

首先，使用[chnroute](http://code.google.com/p/chnroutes/wiki/Usage)配置openvpn，让访问国内网络不走vpn的链路。

使用chnroute的android配置文件即可

>	python chnroute.py -p android

这样得到了`vpnup.sh`和`vpndown.sh`文件，并且把下载下来的文件中前面的`alias`内容注释掉，因为在openwrt中不需要这些配置，会导致openvpn启动错误。

将这两个文件上传到路由器的目录中，并将文件增加执行权限

>	chmod a+x vpnup.sh && chmod a+x vpndown.sh

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