---
layout: post
title: "apt-get error"
description: "an error occured when using apt-get"
category: "linux"
tags: ["ubuntu", "linux"]
---

今天在安装软件时遇到了apt-get发生的一些问题，导致无法安装成功，错误如下：

	E:Encountered a section with no Package: header 
	E:Problem with MergeList /var/lib/apt/lists/cn.archive.ubuntu.com_ubuntu_dists_natty_main_binary-i386_Packages
	...

解决方法如下：

	sudo rm /var/lib/apt/lists/* -vf
	sudo apt-get update

最后耐心等待即可。
{% include JB/setup %}
