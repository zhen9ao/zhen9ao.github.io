---
layout: post
title: "路由器配置vpn的方法"
date: 2011-11-16 22:19
comments: true
published: true
categories: 
---
##  1. 路由器的选择

 才刷坏了一个TP-Link的路由器，不想再用TP-Link或者D-Link的东西了，linksys又偏贵，这时发现了一个叫Buffalo的牌子，网上查了一下，看到这是一个日本品牌，而且比什么Link都要大牌些，[链接在这里](http://net.yesky.com/359/11138359.shtml/)。而且在[DD-WRT](http://www.dd-wrt.com/site/support/router-database/)的数据库中查到对Buffalo的支持，于是在[京东]（http://www.360buy.com/product/138609.html/）败了一个，购买时候的价格是129元。

##  2. 开刷

 我这款[WHR-G300NV2](http://dd-wrt.com/wiki/index.php/WHR-G300N_V2)的官方wiki在这里，按着安装说明里给的链接，找到相关的升级包，从路由器的升级窗口进行升级，方便快捷。刷路由器的时候，最好是用有线的方式进行连接。

##  3. 在路由器中加入自动vpn拨号功能

 在网上查到了一款叫做[autoddvpn](http://code.google.com/p/autoddvpn/)的软件，是用来配置国外ip走vpn，国内ip不走vpn的，也是fork自[chnroutes](http://code.google.com/p/chnroutes/)，相应比较熟悉，也给了比较详细的配置说明文档。

 我这款路由器，虽然有8M的存储空间，但是刷进去的ddwrt还是不支持jsff2，所以不能用graceMode进行配置，只能使用classicMode中的wget方式，这种方式是配置一个iptables规则，让路由器在每次启动联网的时候，自动去相应的地址下载up和down的规则文件，放到/tmp目录中(因为是/tmp，所以每次重启路由器都要重新获取文件)。
  不管用什么方式，都要先配置路由器的vpn功能，配置方法在[这里](http://code.google.com/p/autoddvpn/wiki/HOWTO)，说的比较详细。

 之后就是配置wget脚本，利用的就是ddwrt的命令执行功能，配置方法[在此](http://code.google.com/p/autoddvpn/wiki/WgetMode)。都配置好了之后，路由器每次重连网络就都会按着配置的规则进行联网了。如何查看系统vpn的连接日志呢，在[这里](http://code.google.com/p/autoddvpn/wiki/DEBUG)，若vpn很久都没连通，那就要看看ddwrt中的vpn设置那一步是否配置成功以及vpn服务器是否正常运作。
