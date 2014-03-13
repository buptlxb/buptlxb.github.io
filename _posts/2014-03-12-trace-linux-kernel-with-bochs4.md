---
layout: post
title: "Trace Linux Kernel With Bochs(4)"
description: "Trace the bootsect.s with bochs"
category: "Linux"
tags: ["linux", "bochs", "os"]
tagline: "trace our own boot section"
---

[上一节](/linux/2014/03/10/trace-linux-kernel-with-bochs3/)中我们使用AT&T汇编重写了bootsect.s，这里我们利用bochs来跟踪我们的程序，加深对一些关键操作的认识和理解。

## 修改Makefile

在原来Makefile的基础上，我们简单地修改一下，将我们重写的bootsect.s放在软盘的第一个扇区，然后将之前打印Hello OS World!的程序(称之为hellosect.s)放在第二个扇区，这样当我们执行完bootsect的代码，跳转到setup section时，就会跳转到hellosect，在屏幕上打印出`Hello OS World!`。

代码如下：

``` makefile

CC = gcc
LD = ld
LDFILE = ld_script.ld
OBJCOPY = objcopy
DUMP = objdump
DUMP_FLAGS = -D -b binary -mi386 -M addr16 -M data16

ASM_SRC = $(wildcard *.s)
ASM_OBJ = $(patsubst %.s, %.o, $(ASM_SRC))
ASM_ELF = $(patsubst %.s, %.elf, $(ASM_SRC))
ASM_BIN = $(patsubst %.s, %.bin, $(ASM_SRC))
ASM_DMP = $(patsubst %.s, %.dmp, $(ASM_SRC))


.PHONY: all clean dump

all: linux.img

dump: $(ASM_DMP)

linux.img: $(ASM_BIN)
	@dd if=bootsect.bin of=$@ bs=512 count=1
	@dd if=hellosect.bin of=$@ seek=1 bs=512 count=1
	@dd if=/dev/zero of=$@ seek=2 bs=512 count=2878

$(ASM_BIN): %.bin: %.elf
	@$(OBJCOPY) -R .pdr -R .comment -R .note -S -O binary $< $@

$(ASM_ELF): %.elf: %.o
	$(LD) $< -o $@ -e c -T$(LDFILE)

$(ASM_OBJ): %.o: %.s
	$(CC) -c $< -o $@


$(ASM_DMP): %.dmp: %.bin
	$(DUMP)  $(DUMP_FLAGS) $< > $@

clean:
	@rm -rf $(ASM_OBJ) $(ASM_ELF) $(ASM_BIN) $(ASM_DMP) linux.img

```

其他文件可以到[这里](https://github.com/buptlxb/kernel-test)下载

## Just Trace It!

#### 1.输入命令开始跟踪之旅 -- `bochs -q -f bochsrc`。

![bochs-start]({{ site.url }}/assets/images/bochs-start.png)

#### 2.在BIOS约定的入口处打断点 -- `vb 0x0:0x7c00`

![bochs-entry-break]({{ site.url }}/assets/images/bochs-entry-break.png)

#### 3.执行到下一下断点处 -- `c`

![bochs-bios-print]({{ site.url }}/assets/images/bochs-bios-print.png)

#### 4.查看当前位置后的10条指令 -- `u/10`

![bochs-10-inst]({{ site.url }}/assets/images/bochs-10-inst.png)

#### 5.查看setup section(此处为hellosect)入口处的10条指令 -- `u/10 0x9020:0x0`

![bochs-9020-inst]({{ site.url }}/assets/images/bochs-9020-inst.png)

#### 6.观察bootsect的自我复制移动

##### 6.1.利用dump文件可知指令`rep movsw %ds:(%si), %es:(%di)`的地址，在此处打断点 -- `vb cs:0x7c17`

![bochs-7c17-break]({{ site.url }}/assets/images/bochs-7c17-break.png)

##### 6.2.运行到此断点，查看内存0x9000:0x0处的内容 -- `u/10 0x9000:0x0`

![bochs-9000-before]({{ site.url }}/assets/images/bochs-9000-before.png)

##### 6.3再执行一条指令（复制一条指令），现查看内存0x9000:0x0处的内容 -- `u/10 0x9000:0x0`

![bochs-9000-after]({{ site.url }}/assets/images/bochs-9000-after.png)

#### 7.利用同样的方式可以观察bootsect加载hellosect的过程

![bochs-hello-load]({{ site.url }}/assets/images/bochs-hello-load.png)

#### 8.获取每磁道扇区数

![bochs-disk-param]({{ site.url }}/assets/images/bochs-disk-param.png)

#### 9.打印"Loading system..."提示信息

![bochs-load-system]({{ site.url }}/assets/images/bochs-load-system.png)

#### 10.检查根设备

![bochs-root-dev]({{ site.url }}/assets/images/bochs-root-dev.png)

#### 11.执行hellosect

![bochs-hello-world]({{ site.url }}/assets/images/bochs-hello-world.png)

#### 

{% include JB/setup %}
