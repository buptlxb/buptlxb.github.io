---
layout: post
title: "Trace Linux Kernel With Bochs(1)"
description: "Trace linux kernel with bochs. Learn the OS setup."
category: "Linux"
tags: ["linux", "bochs", "os"]
tagline: "Environment setup"
---

	操作系统是一个神秘的东西，希望我和你都能因为这个博客对它有个一知半解。以下所写均是针对IA32进行的叙述,内核版本为0.11，如有错误，敬请指正。

## BIOS--创世者的功劳

#### 简单的理解

BIOS(`Basic Input/Output System`) 是大部分人都会说的一个词，说起它的作用，也都像我这样的菜鸟也知道它进行一些硬件的检查然后启动操作系统。
至于它如何启动操作系统，就不太清楚了。最近总算又比原来多了点一知半解。

#### 开机加电后的样子

当我们按下电源按键或复位(Reset)按键时，计算机通过硬件的逻辑强行将CS:IP置为`0xf000:fff0`，除些之外，可以想到很多控制寄存器的值也被重置了，比如控制cpu运行在16位实模式下的寄存器位要置位等。这时的cpu工作在实模式下，所谓实模式是与保护模式对应的，也就是内存没有进行分页管理，页表、权限一类的东西都不存在。此时cpu字长是16，地址总线宽度为20。此时的cpu只能访问有限的1MB空间。而BIOS就被映射在这1MB空间的高地址的位置，当CS:IP被重置后，cpu开始执行BIOS中0xffff0位置的指令，这也是cpu执行的内存中的第一条指令。

#### “鸡生蛋，蛋生鸡”的解法

一般认为BIOS是一个`ROM`，当它被映射到地址空间的高处时，cpu访问CS:IP指向的指令时，在cpu看来BIOS与内存，并没有区别。这就是创世者的功劳，解决了“鸡生蛋，蛋生鸡”的难题，也就是如何在掉电就丢失所有内容的内存(`RAM`)中，在加电开后，有指令可以执行的问题。解决的思路异常简单，就是找一个掉电不丢失内容的“内存”，它就是BIOS。

#### BIOS的由来

当然，创世者并不是这么容易就当的，BIOS还有很多工作要做，至少它应该把真正的操作系统的代码(一部分），从硬盘(软盘)读到内存中(至于为什么一定要读到内存中，这怕是要归咎(归功)于冯.诺依曼了)。从外设中读写数据，BIOS应该是由此得名的。BIOS在1MB地址空间的从0开始的1KB空间中写入`中断向量表`，并在其后的适当位置写入`中断服务程序`，这样折腾的原因，无非是与cpu约定如此，否则也没必要在内存中搬来搬去（后面还可以看到这种情形）。有了中断，BIOS执行了`INT 0x19`，将硬盘（软盘）的第一个扇区的512字节，读到`0x07c00`位置上。并跳转到这里，开始执行操作系统的代码。

## Bochs--虚拟的力量

`bochs`是一种x86虚拟机，用来调试操作系统是个不错的选择，写`qemu`相比，它更小巧，同样也是开源免费，更多的内容可以到[wikipedia](http://en.wikipedia.org/wiki/Bochs)去了解。

#### 安装

对于ubuntu用户，直接使用`sudo apt-get install bochs`即可完成安装，对于安装中出现的问题，google一下，大部分都能解决，不能解决的我也解决不了。。。
这样安装虽然简单易用，但安装的bochs是不带debug功能的。

所以我个人推荐下载源码自己编译安装，在[这里](http://sourceforge.net/projects/bochs/files/bochs/)可以找到bochs的源码。我下载的是bochs-2.6.2.tar.gz，安装方法也很简单。

+ 解压源码包
 	
	`tar -xzvf bochs-2.6.2.tar.gz`

+ 进入源码目录，定制

	`cd bochs-2.6.2/;./configure --enable-debugger --enable-disam --with-all-libs`

+ 编译,安装

	`make && sudo make install`

+ 运行

	`bochs`

若出现如下内容说明安装成功：
	![bochs运行]({{ site.url }}/assets/images/bochs-test.png)


{% include JB/setup %}
