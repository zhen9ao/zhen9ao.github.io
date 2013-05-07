---
layout: post
title: "各种代理设置"
date: 2013-05-07 13:43
comments: true
categories: Mac Proxy
---
## Mac中配置Squid

首先，用`Homebrew`安装Squid

	brew install squid
	
启动Squid

	launchctl load /usr/local/opt/squid/*.plist

这样就可以使用你的本机地址+3128端口使用http和https代理了。
	
关闭Squid

	launchctl unload /usr/local/opt/squid/homebrew.mxcl.squid.plist

## Ubuntu配置apt代理

修改配置文件`/etc/apt/apt.conf.d/80proxy`，添加如下内容：

	Acquire::http::Proxy "http://your-ip-addr:3128/";
	


## Ubuntu配置系统全局代理

修改配置文件`/etc/environment`，添加http及https代理：

	http_proxy=http://your-ip-addr:3128/
	https_proxy=http://your-ip-addr:3128/
	
这样配置以后支持http代理的软件都可以访问外网了，不支持的则不行，这时候就需要全局的socket代理了。

## Ubuntu配置socket代理，使用proxychains进行全局配置

前面几种方式都无法正常访问网络，那就需要最后这种方法了。

安装proxychains：

	sudo apt-get install proxychains
	
修改配置文件`/etc/proxychains.conf`，只需要修改其中的`ProxyList`，添加本地的ssh代理即可，如：

	socks5 127.0.0.1 7777
	
使用proxychains，比如要进行bundler安装：

	sudo -u git -H bundle install --without development test postgres --deployment
	
修改为：
	
	sudo -u git -H proxychains bundle install --without development test postgres --deployment
	
bundler就可以使用ssh隧道代理进行更新了。总之就是在要执行的命令前键入一个proxychains就可以使用代理了。