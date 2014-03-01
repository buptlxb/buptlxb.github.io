---
layout: post
title: "Connect to vpn with ppp"
description: ""
category: 
tags: ["linux", "network", "vpn"]
tagline: on the Ubuntu 12.04
---

	很多时候，我们都需要用VPN，当然如果你问我：“那是什么，能吃吗？”，如果你喜欢卖萌，那你可以选择继续看下去，如果你是认真的，最后也认真；google一下VPN是什么。

##图形界面版
	
找到下面的界面，无脑配置即可，方便快捷。
	![Network Manager VPN]({{ site.url }}/assets/images/network-manager-vpn.png)

##命令行版

使用pptp可以在linux下方便地连接到VPN,利用如下命令可以方便地创建VPN连接配置：

	`sudo pptpsetup --create VPN_NAME --server SERVER_IP --username NAME --password PASSWD --encrypt`

然后使用命令`sudo pon VPN_NAME`或`sudo poff VPN_NAME`来开启或关闭vpn连接

利用命令`plog -f`查看vpn连接情况

vpn连接成功后，并不能直接使用，而是需要调整路由表，具体如下：

	`sudo route add -net NET_IP netmask NETMASK dev ppp0`

##更Geek点

参见[Geek](http://www.jiangmiao.org/blog/1914.html)

{% include JB/setup %}
