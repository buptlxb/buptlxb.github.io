---
layout: post
title: "Stack Switching During OS Setup"
description: "A brief introduction of stack switching during OS setup"
category: "Linux"
tags: ["linxu", "os", "bochs"]
tagline: "linux 0.11"
---

	Stack switching might be a frustrating problem during OS setup.In linux 0.11,
	It is easy to find that there are more than one stack defined during the time of OS setup.
	Here is a simple experiment, which indicates that the stacks are defined for different purposes.

## 1. Stack Definition

#### 1.1. The Definition of user_stack

The `user_stack` appears at `line 23` in `boot/head.s` as follow:

```asm

.text
.globl _idt,_gdt,_pg_dir,_tmp_floppy_area
_pg_dir:
startup_32:

	...

	lss _stack_start,%esp
	call setup_idt

	...

```

The definition of `stack_start` can be found at `line 69` in `kernel/sched.c` as follow:

```c

long user_stack [ PAGE_SIZE>>2 ] ;

struct {
	long * a;
	short b;
	} stack_start = { & user_stack [PAGE_SIZE>>2] , 0x10 };

```

It is not difficult to understand the struct `stack_start`:

* `long* a` indicates `ESP`.
* `short b` indicates `SS`.

---

#### 1.2. The definition of Privilege 0 stack

The definition of `Privilege 0 stack` can be found at `line 53` in `kernel/sched.c` as follow:

```c

union task_union {
	struct task_struct task;
	char stack[PAGE_SIZE];
};

static union task_union init_task = {INIT_TASK,};

```

The macro `INIT_TASK` can be found at `line 113` in `include/linux/sched.h` as follow:

```c

/*
 *  INIT_TASK is used to set up the first task table, touch at
 * your own risk!. Base=0, limit=0x9ffff (=640kB)
 */
#define INIT_TASK \
/* state etc */	{ 0,15,15, \
/* signals */	0,{ {},},0, \
/* ec,brk... */	0,0,0,0,0,0, \
/* pid etc.. */	0,-1,0,0,0, \
/* uid etc */	0,0,0,0,0,0, \
/* alarm */	0,0,0,0,0,0, \
/* math */	0, \
/* fs info */	-1,0022,NULL,NULL,NULL,0, \
/* filp */	{NULL,}, \
	{ \
		{0,0}, \
/* ldt */	{0x9f,0xc0fa00}, \
		{0x9f,0xc0f200}, \
	}, \
/*tss*/	{0,PAGE_SIZE+(long)&init_task,0x10,0,0,0,0,(long)&pg_dir,\
	 0,0,0,0,0,0,0,0, \
	 0,0,0x17,0x17,0x17,0x17,0x17,0x17, \
	 _LDT(0),0x80000000, \
		{} \
	}, \
}

```

The `tss_struct` is defined at `line 51` in `include/linux/sched.h` as follow:

```c

struct tss_struct {
	long	back_link;	/* 16 high bits zero */
	long	esp0;
	long	ss0;		/* 16 high bits zero */
	long	esp1;
	long	ss1;		/* 16 high bits zero */
	long	esp2;
	long	ss2;		/* 16 high bits zero */
	long	cr3;
	long	eip;
	long	eflags;
	long	eax,ecx,edx,ebx;
	long	esp;
	long	ebp;
	long	esi;
	long	edi;
	long	es;		/* 16 high bits zero */
	long	cs;		/* 16 high bits zero */
	long	ss;		/* 16 high bits zero */
	long	ds;		/* 16 high bits zero */
	long	fs;		/* 16 high bits zero */
	long	gs;		/* 16 high bits zero */
	long	ldt;		/* 16 high bits zero */
	long	trace_bitmap;	/* bits: trace 0, bitmap 16-31 */
	struct i387_struct i387;
};

```

It is not difficult to figure out the initializers of `esp0` and `ss0`, which describe the Privilege 0 stack.

---

## 2. Simple Experiment

System Info:

	Linux ictlxb-Zhaoyang-E49 3.2.0-60-generic #91-Ubuntu SMP Wed Feb 19 03:54:44 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux

#### 2.1. The Address of user_stack

![the address of user_stack]({{ site.url }}/assets/images/stack_addr.png)

It is easy to know that

* stack_start.a = 0x0001de00
* stack_start.b = 0x0010

#### 2.2. The SS and ESP after lsll

![the ss and esp after lsll]({{ site.url }}/assets/images/sreg_and_reg.png)

It is easy to find that

* ESP = 0x0001de00
* SS = 0x10

#### 2.3. The Current Stack Content(user_stack)

![the current stack content]({{ site.url }}/assets/images/stack.png)

#### 2.4. The Stack Content Before Jumping to 'main'

The code follow is a part of `boot/head.s`.

```asm
	
	...

	pushl $0		# These are the parameters to main :-)
	pushl $0
	pushl $0
	pushl $L6		# return address for main, if it decides to.
	pushl $_main
	jmp setup_paging
L6:
	jmp L6			# main should never return here, but
				# just in case, we know what happens.

	...

```

![the stack content before jumping to main]({{ site.url }}/assets/images/stack_before_main.png)

It is easy to know that

* `_main` stands for address 0x0000670d
* `L6` stands for address 0x00005412

#### 2.5. The Stack Content After 'ret' to 'main'

![the stack content after ret to main]({{ site.url }}/assets/images/ret_to_main.png)

#### 2.6. The Stack Content After Saving Callee-save Registers

According to the calling convension, the registers (`ebp`, `ebx`, `edi`, `esi`) should be saved in the sub-routine.

![the stack content after saving calle-save registers]({{ site.url }}/assets/images/stack_after_save_register.png)

#### 2.7. The Stack Content Before Call 'mem_init'

![the stack content before call mem_init]({{ site.url }}/assets/images/stack_before_call.png)

#### 2.8. The Stack Before 'iret' in 'move_to_user_mode'

![the stack content before iret]({{ site.url }}/assets/images/stack_before_iret.png)

#### 2.9. The Registers After 'iret'

![the registers after iret]({{ site.url }}/assets/images/reg_after_iret.png)

#### 2.10. The Stack Content After 'iret'

![the stack content after iret]({{ site.url }}/assets/images/stack_after_iret.png)

#### 2.11. The Registers Before 'int 0x80'

![the registers before int]({{ site.url }}/assets/images/reg_before_int.png)

#### 2.11. The Stack Content Before 'int 0x80'

![the stack content before int]({{ site.url }}/assets/images/stack_before_int.png)

#### 2.12. The Registers After 'int 0x80'

![the registers after int]({{ site.url }}/assets/images/reg_after_int.png)

#### 2.13. The Stack Content After 'int 0x80'

![the Stack Content after int]({{ site.url }}/assets/images/stack_after_int.png)

---

### _Notice: 2.12 and 2.13 shows the registers change and stack switch!!!_

## Conclusion

1. The `user_stack` is used by kernel before Process 0 `move to user mode`.

2. The `user_stack` is used as `Privilege 3 stack` (so-called `user stack` of Porcess 0) by Process 0 after it `move to user mode`.

3. When Process 0 executes `int 0x80`, the hardware will switch stack to `Privilege 0` stack (so-called `kernel stack` of Porcess 0) along with Privilege changing from `3` to `0`.

4. The Process 0 and Process 1 share the `user_stack` as their `user stack`.

{% include JB/setup %}
