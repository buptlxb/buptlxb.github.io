---
layout: post
title: "Trace Linux Kernel With Bochs(5)"
description: "rewrite setup.s"
category: "Linux"
tags: ["linux", "bochs", "os"]
tagline: "rewrite setup.s"
---

	当bootsect.s成功执行完成时，它已经将setup.s加载到了内存中(0x90200)的位置上，也将内核代码加载到了内存中(0x10000)。它的生命就随着setup.s的执行走到了尽头，但我们知道它曾来过。

下面，我们一边介绍setup.s的功能，一边重写代码。

## 为内核运行准备数据

这些数据的获得大多来自由BIOS提供的中断，在关中断之前，我们必须将这些与机器硬件密切相关的数据保存起来，而保存的位置，就是从`0x90000`开始的一段内存，如果你看过前面的几篇博文，应该知道这里曾是bootsect.s的位置。

#### 获取光标位置

我们首先获取光标的位置,将它存储在0x90000开始的２个字节中。

``` asm

	movb $0x03, %ah		# read cursor pos
	xor %bh, %bh
	int $0x10			# save it in known place, con_init fetches
	movw %dx, 0	# it from 0x90000

```
#### 获取扩展内存的大小

扩展内存指的是1M 以上的物理内存大小，单位是kB。存储在0x90002开始的２个字节中。

```asm

	movb $0x88, %ah
	int $0x15
	movw %ax, 2

```

#### 获取显示的相关信息

获取显示相关信息，如当前显示模式，窗口宽度等。存储在0x90004开始的４个字节中。

```asm

	movb $0x0f, %ah
	int $0x10
	movw %bx, 4		# BH = display page
	movw %ax, 6		# AL = video mode, AH = window width

```

#### 获取EGA/VGA相关信息

存储在0x90008开始的６个字节中。

```asm

	movb $0x12, %ah
	movb $0x10, %bl
	int $0x10
	movw %ax, 8
	movw %bx, 10
	movw %cx, 12

```

#### 获取第一个硬盘的信息

存储在0x90080开始的16个字节中。

```asm

	xor %ax, %ax
	movw %ax, %ds
	lds 0x41*4, %si	
	movw $INITSEG, %ax
	movw %ax, %es
	movw $0x0080, %di
	movw $0x10, %cx
	rep
	movsb

```

#### 获取第二个硬盘的信息

存储在0x90090开始的16个字节中。

```asm

	xor %ax, %ax
	movw %ax, %ds
	lds 0x46*4, %si	# what is this? why is the address?
	movw $INITSEG, %ax
	movw %ax, %es
	movw $0x0090, %di
	movw $0x10, %cx
	rep
	movsb
```
#### 检测是否存在第二个硬盘

```asm

	movw $0x1500, %ax
	movb $0x81, %dl
	int $0x13
	jc no_disk1
	cmp $3, %ah
	je is_disk1
no_disk1:
	movw $INITSEG, %ax
	movw %ax, %es
	movw $0x0090, %di
	movw $0x10, %cx
	xor %ax, %ax
	rep
	stosb
is_disk1:

```

｀在前面无聊的取数存数后，我们将进行一项意义深远的举动，我们将不再依赖BIOS提供的中断，我们将开始为内核启动做最后的准备，时间貌似不多了，Let us roll`

## 关中断

```asm

cli

```

## 搬内核

我们将系统代码从0x10000的位置，搬到0x00000开始的位置，在这个过程中，我们废弃了BIOS在内存低1k位置准备的中断向量表，当然也废弃了BIOS准备的中断服务程序。

```asm
	
	xor %ax, %ax
	cld		# 'direction' = 0, 'movs' moves forward

do_move:
	movw %ax, %es		# destination segment
	add $0x1000, %ax
	cmp $0x9000, %ax
	jz end_move
	movw %ax, %ds		# source segment
	xor %di, %di
	xor %si, %si
	movw $0x8000, %cx
	rep
	movsw
	jmp do_move

```

## 启用新的中断描述符表

设置中断描述符表和全局描述符表。在x86体系结构中，开启保护模式后，中断向量表将不再必须放在内存低１k的位置，而是采用了寄存器`idtr`来指向中断描述符表在内存中的位置。同时我们也设置了`gdtr`，稍后我们会解释它的作用。

```asm

end_move:
	movw $SETUPSEG, %ax 
	movw %ax, %ds
	lidt idt_48		# load idt with 0,0
	lgdt gdt_48		# load gdt with whatever appropriate

```

## 进入32位的世界

从这开始，我们打开A20,使用32位地址总线，进入32位的世界。

```asm

	call empty_8042
	movb $0xD1, %al		# command write
	outb %al, $0x64
	call empty_8042
	movb $0xDF, %al		# A20 on
	outb %al, $0x60
	call empty_8042

```

## 重新设置8259A

8259A芯片是中断控制器，之所以说是重新设置，是因为在此之前，它已经被BIOS设置过一次，为了使用BIOS的中断。而现在，重新设置只是因为要符合Intel的要求。

```asm

	movb $0x11, %al		# initialization sequence
	out	%al, $0x20		# send it to 8259A-1
	.word	0x00eb,0x00eb		# jmp $+2, jmp $+2
	out	%al, $0xA0		# and to 8259A-2
	.word	0x00eb,0x00eb
	mov	$0x20, %al		# start of hardware int's (0x20)
	out	%al, $0x21
	.word	0x00eb,0x00eb
	mov	$0x28, %al		# start of hardware int's 2 (0x28)
	out	%al, $0xA1
	.word	0x00eb,0x00eb
	mov	$0x04, %al		# 8259-1 is master
	out	%al, $0x21
	.word	0x00eb,0x00eb
	mov	$0x02, %al		# 8259-2 is slave
	out	%al, $0xA1
	.word	0x00eb,0x00eb
	mov	$0x01, %al		# 8086 mode for both
	out	%al, $0x21
	.word	0x00eb,0x00eb
	out	%al, $0xA1
	.word	0x00eb,0x00eb
	mov	$0xFF, %al		# mask off all interrupts for now
	out	%al, $0x21
	.word	0x00eb,0x00eb
	out	%al, $0xA1

```

## 开启保护模式

经过一系列的准备工作之后，我们打开保护模式，这是一个与实模式相对的概念。

```asm

	mov	$0x0001, %ax	# protected mode (PE) bit
	lmsw	%ax			# This is it!
	ljmp	$8, $0			# jmp offset 0 of segment 8 (cs)

```

在我们ljmp之前，有些事情要说清楚：

* gdtr 和 gdt

gdtr是一个48位置的系统地址寄存器，保存了全局描述符表(gdt)的基址(32位)和限长(16位)

gdt则是一个保存了段相关信息的数组，每个元素的大小为64 bit。

关于段机制的详细内容可以参看[资料](http://oss.org.cn/kernel-book/ch02/2.3.1.htm)
关于lgdt和lidt命令的详细内容可以参看[资料](http://pdos.csail.mit.edu/6.828/2008/readings/i386/LGDT.htm)

* ljmp $8, $0

在打开A20和开启保护模式后,段寄存器的用法发生了改变:

在16位模式时，段寄存器保存的是段基址。

而在32位保护模式时，显然无法将32位的地址保存在16位的寄存器中，又由于要保持良好的兼容性，段寄存器中改为保存16位的段选择子，用来选取gdt或ldt中的某一个段描述符。

这里$8的低２位是RPL(Request Privilege Level)，第三位是使用gdt还是ldt，高13位是gdt或ldt数组的index。

## 数据和辅助函数

```asm

INITSEG  = 0x9000	# we move boot here - out of the way
SYSSEG   = 0x1000	# system loaded at 0x10000 (65536).
SETUPSEG = 0x9020	# this is the current segment

empty_8042:
	.word	0x00eb,0x00eb
	in	$0x64, %al	# 8042 status port
	test $2, %al	# is input buffer full?
	jnz	empty_8042	# yes - loop
	ret

gdt:
	.word	0,0,0,0		# dummy

	.word	0x07FF		# 8Mb - limit=2047 (2048*4096=8Mb)
	.word	0x0000		# base address=0
	.word	0x9A00		# code read/exec
	.word	0x00C0		# granularity=4096, 386

	.word	0x07FF		# 8Mb - limit=2047 (2048*4096=8Mb)
	.word	0x0000		# base address=0
	.word	0x9200		# data read/write
	.word	0x00C0		# granularity=4096, 386

idt_48:
	.word	0			# idt limit=0
	.word	0,0			# idt base=0L

gdt_48:
	.word	0x800		# gdt limit=2048, 256 GDT entries
	.word	512+gdt,0x9	# gdt base = 0X9xxxx

.org 2048

```
{% include JB/setup %}
