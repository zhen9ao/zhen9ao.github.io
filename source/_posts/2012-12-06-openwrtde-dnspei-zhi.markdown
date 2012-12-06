---
layout: post
title: "OpenWrt的DNS配置"
date: 2012-12-06 21:30
comments: true
categories: 
---

本文主要研究一下OpenWrt中的DNS配置的相关内容，介绍以下内容：

1. dnsmasq的更多配置，给域名设置独立的DNS server。
2. 用pdnsd配置一个更安全、快速的DNS server。

##dnsmasq

国外域名用Google DNS进行解析还好说，国内的域名如果用Google DNS进行解析，可能会出现返回的服务器IP不是离你最近的情况，导致服务器响应没那么快，比较常见的是Youku等视频网站。所以针对这种情况，针对国内的域名，配置国内的DNS服务。

<!--more-->

需要进行配置的文件就是`/etc/dnsmasq.conf`，可以把所有的域名配置都写到这个文件里，也可以像我一样新建一个配置文件夹如`/home/vpn/dnsmasq.d`，在这个文件夹中放入`china.conf`文件。而在`/etc/dnsmasq.conf`中添加如下配置：

	config-dir=/home/vpn/dnsmasq.d
	
具体的`china.conf`中的DNS server配置如下：

{% gist 4224532 china.conf %}

这样重启dnsmasq，国内的一些域名就会使用所配置的DNS Server了。

##pdnsd

最近DNS污染严重，在使用`dnsmasq`的时候发现经常出现查询结果被污染的情况，而且返回的DNS解析结果缓存TTL时间相当长，虽然`dnsmasq`可以用TCP连接Google DNS，但是由于不能修改默认的缓存TTL时间，一般的TTL时间是20几秒，也就是说20几秒过后，要是再对刚才的域名进行请求，还要重新向Google DNS发起查询请求，这一方面会导致网络延迟加大，另一方面频繁请求，DNS cache被污染的几率就加大。

`pdnsd`可以实现设置TTL的目的，也可以设置向DNS Server的请求为TCP连接，比较安全。

在OpenWrt中，要先安装
	
	opkg install pdnsd
	
是时候禁用dnsmasq的服务了，不过不能stop，因为dnsmasq还作为wifi的DHCP server来用，所以只能禁用DNS Server的功能，保留DHCP server。在`/etc/config/dhcp`中的`config dnsmasq`中添加如下配置：
	
	option port '0'
	
重启`dnsmasq`服务，这样就禁用了DNS Server的功能。

然后配置`pdnsd`，配置文件是`/etc/pdnsd.conf`，主要修改其中的`global`和`server`配置项，修改后如下(下面给出的是要修改的部分)：

    global {
        server_ip=any;
        query_method=tcp_only;
        min_ttl=1d;
    }

    server {
        ip=8.8.8.8,8.8.4.4;
        uptest=ping;
    }

这样，就可以设置pdnsd开机启动了：
    
    /etc/init.d/pdnsd enable
    /etc/init.d/pdnsd start


PS: 本文主要参考了：[用PDNSD + Google DNS 获得高速正确的dns解析](http://bullshitlie.blogspot.jp/2012/03/pdnsd-google-dns-dns.html)
