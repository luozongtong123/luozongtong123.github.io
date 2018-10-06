野火出了本书 ————《RT-Thread 内核实现与应用开发实战指南》，这本书基于 STM32 讲解了 RT-Thread 内核的实现以及 RT-Thread 内核应用开发。现在结合这本书学习一下 RT-Thread。这是这一系列学习笔记的第一篇。
<!--more-->

电子版的书和工程源文件以及参考资料下载链接如下：   

链接: [点击直达](https://pan.baidu.com/s/1VxToy8wi6Lg5ozhrNqgXRA)  提取码: mqrb



# 裸机与多线程



最开始学习单片机的时候大家都是直接跑裸机不使用操作系统，裸机的优势是不需要学习操作系统的知识，当实现一个简单的程序时裸机简单快捷。当系统的 Flash 和 ROM 资源紧张时，裸机可以节省一部分资源。   

## 裸机系统

裸机程序分为轮询系统和加入中断的前后台系统。轮询系统就是程序在主函数中有一个死循环不断执行某些任务。轮询系统只能被动有延时地处理外界对系统的影响无法主动实时地对外界变化多处反应。为了提供系统的响应实时性，引入了中断机制。   

回顾一下中断机制：CPU 在执行每一条指令后会确认刚才执行指令期间是否有中断请求过来，如果有中断请求则 CPU 会中断当前程序的执行，保存程序的现场（一些寄存器压栈 push），然后跳转到对应的中断服务函数并执行中断服务函数，中断服务函数执行完毕后出栈，恢复程序的执行。   

引入了中断机制的轮询系统称为前后台系统，其中中断执行的过程为前台，主函数中的轮询为后台。前后台系统确保了外部事件不会丢失并能够得到及时的响应。在大多数中小型系统中前后台系统应用的得当的话可以得到堪比加入了操作系统的效果。   

相比于前后台系统，多线程系统对外部事件的响应也是在中断中完成的，但是事件的处理是在线程中完成的。在多线程系统中，线程跟中断一样，也具有优先级，优先级高的线程会被优先执行。当一个紧急的事件在中断被标记之后，如果事件对应的线程的优先级足够高，就会立马得到响应。相比前后台系统，多线程系统的实时性又被提高了。   



## 多线程系统

在多线程系统中，我们根据不同的功能将程序分为多个小的独立的无限循环的任务，这个任务称为线程。每个线程独立、互不干扰、且拥有自己的优先级，线程由实时操作系统进行调度的管理。加入操作系统后我们就不需要精心设计程序流，只需要设计好各个进行的逻辑和不同进程间的通信即可，对大型系统而言，加入了操作系统会使系统的可维护性更好。   

关于上不上操作系统从来就不是一个确定的问题，这需要根据实际的需求来决定。如果只是一个简单的系统，强行加入操作系统只会增加开发和维护的难度。   



# 线程与线程的创建

前面讲到，在裸机系统中，系统的主体就是 main 函数里面顺序执行的无限循环，这个无限循环里面 CPU 按照顺序完成各种事情。在多线程系统中，我们根据功能的不同，把整个系统分割成一个个独立的且无法返回的函数，这个函数我们称为线程。

## 定义线程栈

在裸机系统中，自动局部变量是存储在栈上的，栈的管理是由 C 语言编译器来完成的，栈的初始化一般是在启动文件中或者是在链接命中指定，然后由 C 语言的 `_main`来完成初始化的操作。逻辑系统中，整个程序只有一个栈，多个函数调用的过程中就对这个栈进行压栈和出栈操作。对于有 RTOS 的系统来言，我们要实现线程间的隔离就需要为每个线程创建独立的栈，在该线程内的函数调用只需要对线程自己的栈进行操作。   

这个由进程使用的栈一般是一个预先定义的一个全局数组或者一段动态分配的内存。   

```c
/* 设置字节对齐 */
ALIGN(RT_ALIGN_SIZE) (2)
/* 定义线程栈 */
rt_uint8_t rt_flag1_thread_stack[512];
rt_uint8_t rt_flag2_thread_stack[512];
```

`rt_uint8_t `是在 `rtdef.h` 中定义的类型，其他会用到的数据类型详见[这里](/src/rt-thread-learning01/rtdef.h)。   

## 定义线程函数

```c
/* 软件延时 */
void delay (uint32_t count)
{
	for(; count!=0; count--);
}

/* 线程1 */
void flag1_thread_entry( void *p_arg )
{
	for( ;; )
	{
		flag1 = 1;
		delay( 100 );		
		flag1 = 0;
		delay( 100 );
		
		/* 线程切换，这里是手动切换 */		
		rt_schedule();
	}
}

/* 线程2 */
void flag2_thread_entry( void *p_arg )
{
	for( ;; )
	{
		flag2 = 1;
		delay( 100 );		
		flag2 = 0;
		delay( 100 );
		
		/* 线程切换，这里是手动切换 */
		rt_schedule();
	}
}

```

如前文所述，线程是一个独立的、无限循环且不能返回的函数。

## 定义线程控制块

线程控制块是一个线程的标识，线程调度器通过现称快隋朝块识别调度线程，线程控制块的定义如下：

```c
struct rt_thread
{
	void        *sp;	          /* 线程栈指针 */
	void        *entry;	          /* 线程入口地址 */
	void        *parameter;	      /* 线程形参 */	
	void        *stack_addr;      /* 线程起始地址 */
	rt_uint32_t stack_size;       /* 线程栈大小，单位为字节 */
	
	rt_list_t   tlist;            /* 线程链表节点 */
};
typedef struct rt_thread *rt_thread_t;
```

在 RT-Thread 中，都会给新声明的数据结构重新定义一个指针。往后如果要定义线程控制块变量就使用  `struct rt_thread xxx` 的形式，定义线程控制块指针就使用 `rt_thread_t xxx` 的形式。



## 线程创建函数的实现 {#section-rt_thread_init}

创建线程就是根据给出的参数如线程的名称、线程栈的大小等完成线程栈的初始化和线程控制块的初始化。线程的创建由函数 `rt_thread_init()` 来实现，该函数在 `thread.c` 中定义在 `rthread.h` 中声明。该函数的实现如下：

```c
rt_err_t rt_thread_init(struct rt_thread *thread,        // 线程控制块
                        void (*entry)(void *parameter),  // 线程函数名即线程入口
                        void             *parameter,     // 线程形参
                        void             *stack_start,   // 线程栈起始地址
                        rt_uint32_t       stack_size)    // 线程栈大小
{
	rt_list_init(&(thread->tlist));                      // (1)
	
	thread->entry = (void *)entry;    // 将线程入口保存到线程控制块的 entry成员中
	thread->parameter = parameter;    // 将线程入口形参保存到线程控制块的 parameter 成员中

	thread->stack_addr = stack_start; // 将线程栈起始地址保存到线程控制块的 stack_start 成员中
	thread->stack_size = stack_size;  // 将线程栈起大小保存到线程控制块的 stack_size成员中
	
	/* 初始化线程栈，并返回线程栈指针 */
	thread->sp = (void *)rt_hw_stack_init( thread->entry, 
		                                   thread->parameter,
							               (void *)((char *)thread->stack_addr + thread->stack_size - 4) );
	
	return RT_EOK;
}
```

标号为 (1) 的代码对线程控制块的线程链表节点 `tlist` 进行初始化，链表节点可以插入到不同的链表，线程调度器通过链表调度线程。`tlist` 的数据原型是 `rt_list_t` 。      

> **此处有个问题，线程控制块链表节点初始化的目的是什么？**

双向链表节点数据类型定义如下：

```c
struct rt_list_node
{
  struct rt_list_node *next;   /* 指向后一个节点 */
  struct rt_list_node *prev;   /* 指向前一个节点 */
};
typedef struct rt_list_node rt_list_t;
```

节点中包含两个数据第一个是指向下一个节点的指针，第二个是指向上一个节点的指针。   

关于这个节点的作用书中在此处没有说明，他的作用是在后面讲的，我们把这部分拿到前面来。前面讲到我们可以把 `tlist` 这个线程控制块的成员插入到不同的链表中，例如就绪列表等。线程调度器再根据列表中的节点切换不同的线程，那么问题来了，我们如何利用这个节点找到线程控制块呢？ RT-Thread 中有一个宏可以通过 `tlist` 来找到线程控制块的起始地址，这个宏是 `rt_list_entry()` ，其定义如下：

```c
/* 已知一个结构体里面的成员的地址，反推出该结构体的首地址 */
#define rt_container_of(ptr, type, member)
    ((type *)((char *)(ptr) - (unsigned long)(&((type *)0)->member)))

#define rt_list_entry(node, type, member)
    rt_container_of(node, type, member)
```

`rt_list_entry()` 调用了 `rt_container_of()` 宏，这个宏可以已知结构体中的成员找到结构体的首地址。上述代码中， `prt` 是指向结构体成员的地址， `type` 是结构体的类型， `member` 是结构体成员的名字。我们来看一下这个宏函数。

{{% figure class="center" src="/img/rt-thread-learning01/rt_container_of.jpg" alt="rt_container_of() 宏的实现" title="rt_container_of() 宏的实现" %}}

如上图左半边图所示，我们已知 `prt` 要求 `f_struct_prt` ，显然只要知道深色部分标注为**偏移**的大小即可。那么如何才能求出偏移量的大小呢，这需要借助于 **0 地址**。

1. 首先，将 0 地址强制转换为 `type` 类型的结构体： `((type *)0)`，

2. 然后，获取到这个结构体的成员 `member` 的地址： `(&((type *)0)->member))`，
3. 最后，因为这个结构体的起始地址是 0 ，所以只要将这个成员的地址强制转换成无符号长整型数就是我们要求的偏移： `(unsigned long)(&((type *)0)->member))`。   

再回到 `(char *)(ptr)` ，这里将指针强制转换为 `char` 的目的是为了在后面加减偏移量时以一个字节为单位。最后的最后，再将减去偏移量的地址强制转换为结构体类型的指针。至于 `rt_list_entry()` ，则可以非常容易的理解为，已知节点地址 `node` 、线程控制块结构体类型 `type` 、节点名称（tlist） `member` ，求线程控制块的地址。      

讲完了节点的作用下面我们来看一下节点双向列表的一些操作。   

由上面给出的节点的定义可知，由节点构成的链表可以表示成下图：

{{% figure class="center" src="/img/rt-thread-learning01/list_of_rt_list_t.jpg" alt="rt_list_t列表" title="rt_list_t列表" %}}

链表的操作包括初始化链表节点、在双向链表表头前面插入一个节点、在双向链表表头后面插入一个节点、从双向链表中删除一个节点。相关函数的实现在 ` rtservice.h` 。   

**初始化链表节点**     

rt_list_t 类型的节点的初始化，就是将节点里面的 next 和 prev 这两个节点指针指向节点本身。代码实现如下：

```c
rt_inline void rt_list_init(rt_list_t *l)
{
    l->next = l->prev = l;
}
```

{{% figure class="center" src="/img/rt-thread-learning01/list_op_1.jpg" alt="初始化链表节点" title="初始化链表节点" %}}   



**在双向链表表头后面插入一个节点**      

> 此处的表头的表述是否有问题，因为从双向链表的结构来看，双向链表没有所谓的表头？   
>
> 是否应该说 向双向链表的 `l` 节点后插入 `n` ？   

插入节点一共需要四步操作，即图示四个红色箭头的操作，将这四步操作转换为代码则是下面的代码中四步。仔细观察这四步操作，不难发现，如果要保证代码完成我们想要的操作，代码执行的前后顺序是有要求的。即：第三步对 `l->next` 的赋值操作要在第一步和第二部对其引用之后。只要保证这个前后关系满足要求即可。   

代码中采用这种顺序实现起来更加 ~~优雅~~ 并不。

```c
/* 在双向链表头部插入一个节点 */
rt_inline void rt_list_insert_after(rt_list_t *l, rt_list_t *n)
{
    l->next->prev = n;   /* 第1步 */
    n->next = l->next;   /* 第2步 */
    
    l->next = n;         /* 第3步 */
    n->prev = l;         /* 第4步 */
}
```



{{% figure class="center" src="/img/rt-thread-learning01/list_op_2.jpg" alt="在双向链表表头后面插入一个节点" title="在双向链表表头后面插入一个节点" %}}   



**在双向链表表头前面插入一个节点**     

不难理解只需要将后插操作代码中的的 `prev` 和 `next` 调换一下就可以完成前插操作。

```c
rt_inline void rt_list_insert_before(rt_list_t *l, rt_list_t *n)
{
    l->prev->next = n;   /* 第1步 */
    n->prev = l->prev;   /* 第2步 */
    
    l->prev = n;         /* 第3步 */
    n->next = l;         /* 第4步 */
}
```

{{% figure class="center" src="/img/rt-thread-learning01/list_op_3.jpg" alt="在双向链表表头前面插入一个节点" title="在双向链表表头前面插入一个节点" %}}      



**从双向链表删除一个节点**     

删除操作较插入操作要简单一些，只需要注意第三步在前两步之后即可。

```c
rt_inline void rt_list_remove(rt_list_t *n)
{
    n->next->prev = n->prev;  /* 第1步 */
    n->prev->next = n->next;  /* 第2步 */
    
    n->next = n->prev = n;    /* 第3步 */
}
```

{{% figure class="center" src="/img/rt-thread-learning01/list_op_4.jpg" alt="从双向链表删除一个节点" title="从双向链表删除一个节点" %}}      

**rt_hw_stack_init()函数**     

[rt_thread_init() 函数](#section-rt_thread_init) 中还用到了 `rt_hw_stack_init()` 函数 ，该函数对线程栈进行初始化，其代码实现如下：

```c
/* 线程栈初始化 */
rt_uint8_t *rt_hw_stack_init(void       *tentry,      // 线程入口
                             void       *parameter,   // 线程形参
                             rt_uint8_t *stack_addr)  // 线程栈顶地址
{
	
	
	struct stack_frame *stack_frame;                  // (1)
	rt_uint8_t         *stk;
	unsigned long       i;
	
	
	/* 获取栈顶指针 rt_hw_stack_init 在调用的时候，传给stack_addr的是(栈顶指针)*/
	stk  = stack_addr + sizeof(rt_uint32_t);
	
	/* 让stk指针向下8字节对齐 */
	stk  = (rt_uint8_t *)RT_ALIGN_DOWN((rt_uint32_t)stk, 8);
	
	/* stk指针继续向下移动sizeof(struct stack_frame)个偏移 */
	stk -= sizeof(struct stack_frame);
	
	/* 将stk指针强制转化为stack_frame类型后存到stack_frame */
	stack_frame = (struct stack_frame *)stk;
	
	/* 以stack_frame为起始地址，将栈空间里面的 sizeof(struct stack_frame) 个内存初始化为0xdeadbeef */
	for (i = 0; i < sizeof(struct stack_frame) / sizeof(rt_uint32_t); i ++)
	{
			((rt_uint32_t *)stack_frame)[i] = 0xdeadbeef;
	}
	
	/* 初始化异常发生时自动保存的寄存器 */
	stack_frame->exception_stack_frame.r0  = (unsigned long)parameter; /* r0 : argument */
	stack_frame->exception_stack_frame.r1  = 0;                        /* r1 */
	stack_frame->exception_stack_frame.r2  = 0;                        /* r2 */
	stack_frame->exception_stack_frame.r3  = 0;                        /* r3 */
	stack_frame->exception_stack_frame.r12 = 0;                        /* r12 */
	stack_frame->exception_stack_frame.lr  = 0;                        /* lr */
	stack_frame->exception_stack_frame.pc  = (unsigned long)tentry;    /* entry point, pc */
	stack_frame->exception_stack_frame.psr = 0x01000000L;              /* PSR */
	
	/* 返回线程栈指针 */
	return stk;
}
```

代码 `(1)` 处定义了一个 `struct stack_frame` 类型的结构体指针 `stack_frame`，该结构体类型在 `cpuport.c` 中定义，具体代码如下：   

```c
struct exception_stack_frame
{
    /* 异常发生时自动保存的寄存器 */
	rt_uint32_t r0;
    rt_uint32_t r1;
    rt_uint32_t r2;
    rt_uint32_t r3;
    rt_uint32_t r12;
    rt_uint32_t lr;
    rt_uint32_t pc;
    rt_uint32_t psr;
};

struct stack_frame
{
    /* r4 ~ r11 register 异常发生时需手动保存的寄存器 */
    rt_uint32_t r4;
    rt_uint32_t r5;
    rt_uint32_t r6;
    rt_uint32_t r7;
    rt_uint32_t r8;
    rt_uint32_t r9;
    rt_uint32_t r10;
    rt_uint32_t r11;

    struct exception_stack_frame exception_stack_frame;
};
```

为了看懂栈的初始化过程，我们首先要明确一个问题：栈是从高地址向低地址生长的，在线程初始化时传进初始化函数的参数是栈的起始地址，即**栈尾地址**，栈的初始化首先需要根据栈尾地址和栈的大小确定**栈首地址**。计算栈首地址在线程初始化函数中调用栈初始化函数时计算并传给了栈初始化函数。线程控制块中的**栈顶地址**是当前栈的位置。

> **有一点不明白的是调用栈初始化函数时减掉了 4，在栈初始化函数中又加了回去，不明白这样处理的意义。**   

接着来看栈初始化函数，得到栈顶地址后，首先进行了向下 8 字节对齐。8 字节对齐的目的是为了后续实现浮点数的运算支持。向下是指当不满足 8 字节对齐时向下移动栈顶指针使其满足对齐要求，例如如果对齐前栈顶地址是 33 ，则对齐后的栈顶地址为 32，即空出一个 33 地址的位置不用。

{{% figure class="center" src="/img/rt-thread-learning01/stk-stack.jpg" alt="完成栈地址字节对齐后的栈空间" title="完成栈地址字节对齐后的栈空间" %}}      

完成栈地址对齐后，接下来要**对首次线程切换时用到的栈中的数据进行初始化**。将栈地址向下移动 `sizeof(struct stack_frame)` 个地址，将 `stk` 指针强制转化为 `stack_frame` 类型后存到指针变量
`stack_frame` 中，这个时候 `stack_frame` 在线程栈里面的指向具体见下图：

{{% figure class="center" src="/img/rt-thread-learning01/stack_frame-stack.jpg" alt="stack_frame 的栈空间" title="stack_frame 的栈空间" %}}    

接下来以 `stack_frame` 为起始地址，将栈空间里面的 `sizeof(struct stack_frame)` 个内存初始化为  `0xdeadbeef ` 。将内存初始化为 `0xdeadbeef ` 的原因是，这个数字是一个在嵌入式领域常用的魔力数，只是用来标识未使用的内存。接下来，设置线程初次运行时加载到 CPU 寄存器的环境参数。从栈顶开始，初始化的顺序固定，首先是异常发生时自动保存的 8 个寄存器，即 xPSR、R15、R14、R12、R3、R2、R1和 R0。其中 xPSR 寄存器的位 24 必须是 1，R15 PC 指针必须存的是线程的入口地址，R0 必须是线程形参，剩下的 R14、R12、R3、R2、 R1我们初始化为 0。完成初始化的栈空间如下图所示：

{{% figure class="center" src="/img/rt-thread-learning01/inited-stack.jpg" alt="初始化完成后的栈空间" title="初始化完成后的栈空间" %}}    

最后栈初始化函数返回线程栈顶指针 `stk` ，线程初始化函数返回线程创建成功的错误码。关于错误码，其定义如下：

```c
/* RT-Thread 错误码重定义 */
#define RT_EOK                          0               /**< There is no error */
#define RT_ERROR                        1               /**< A generic error happens */
#define RT_ETIMEOUT                     2               /**< Timed out */
#define RT_EFULL                        3               /**< The resource is full */
#define RT_EEMPTY                       4               /**< The resource is empty */
#define RT_ENOMEM                       5               /**< No memory */
#define RT_ENOSYS                       6               /**< No system */
#define RT_EBUSY                        7               /**< Busy */
#define RT_EIO                          8               /**< IO error */
#define RT_EINTR                        9               /**< Interrupted system call */
#define RT_EINVAL                       10              /**< Invalid argument */

```

最后，回顾一下主函数中，调用线程初始化函数进行线程初始化的代码：

```c
int main(void)
{	
	/* 硬件初始化 */
	/* 将硬件相关的初始化放在这里，如果是软件仿真则没有相关初始化代码 */	

	/* 初始化线程 */
	rt_thread_init( &rt_flag1_thread,                 /* 线程控制块 */
	                flag1_thread_entry,               /* 线程入口地址 */
	                RT_NULL,                          /* 线程形参 */
	                &rt_flag1_thread_stack[0],        /* 线程栈起始地址 */
	                sizeof(rt_flag1_thread_stack) );  /* 线程栈大小，单位为字节 */

	
	/* 初始化线程 */
	rt_thread_init( &rt_flag2_thread,                 /* 线程控制块 */
	                flag2_thread_entry,               /* 线程入口地址 */
	                RT_NULL,                          /* 线程形参 */
	                &rt_flag2_thread_stack[0],        /* 线程栈起始地址 */
	                sizeof(rt_flag2_thread_stack) );  /* 线程栈大小，单位为字节 */

}
```



# 实现就绪列表

**定义就绪列表**   

线程创建完成之后，我们需要把线程添加到就序列表，表示线程已经就绪，系统随时可以调度。就绪列表在 ` scheduler.c` 中定义，其定义如下：

```c
rt_list_t rt_thread_priority_table[RT_THREAD_PRIORITY_MAX]; 
```

就绪列表实际上就是一个 rt_list_t 类型的数组，数组的大小由决定最大线程优先级的宏 `RT_THREAD_PRIORITY_MAX` 决定 ，`RT_THREAD_PRIORITY_MAX` 在 rtconfig.h 中默认定义为 32。数组的下标对应了线程的优先级，同一优先级的线程统一插入到就绪列表的同一条链表中。一个空的就绪列表图下图：

{{% figure class="center" src="/img/rt-thread-learning01/priority_table.jpg" alt="一个空的就序列表" title="一个空的就序列表" %}}    



**将线程插入到就绪列表**   

线程控制块中有一个类型为 `rt_list_t` 的成员 `tlist` ，将线程插入到就绪列表就是利用之前实现的链表操作函数将 `tlist` 插入到就序列表里面。具体实现代码如下：

```c
/* 初始化线程 */
rt_thread_init( &rt_flag1_thread,                 /* 线程控制块 */
	            flag1_thread_entry,               /* 线程入口地址 */
	            RT_NULL,                          /* 线程形参 */
	            &rt_flag1_thread_stack[0],        /* 线程栈起始地址 */
	            sizeof(rt_flag1_thread_stack) );  /* 线程栈大小，单位为字节 */
/* 将线程插入到就绪列表 */
rt_list_insert_before( &(rt_thread_priority_table[0]),&(rt_flag1_thread.tlist) );
	
/* 初始化线程 */
rt_thread_init( &rt_flag2_thread,                 /* 线程控制块 */
	            flag2_thread_entry,               /* 线程入口地址 */
	            RT_NULL,                          /* 线程形参 */
	            &rt_flag2_thread_stack[0],        /* 线程栈起始地址 */
	            sizeof(rt_flag2_thread_stack) );  /* 线程栈大小，单位为字节 */
/* 将线程插入到就绪列表 */
rt_list_insert_before( &(rt_thread_priority_table[1]),&(rt_flag2_thread.tlist) );
```

就绪列表的下标就是线程的优先级，目前还没有实现优先级，有关优先级的内容后面会讲到。目前，只是将 flag1 的线程插入到下标为 0 的链表，将 falsg2 的线程插入到下标为 1 的链表。具体的示意图如下：

{{% figure class="center" src="/img/rt-thread-learning01/priority_table_insert.jpg" alt="插入线程的就序列表" title="插入线程的就序列表" %}}    



# 实现调度器

线程调度器是实时操作的核心，其主要功能就是从就序列表中找到优先级最高的线程并切换到这个线程。从代码上来看，线程调度器是由一些全局变量和一些可以实现线程切换的函数组成。线程调度器的代码在 `scheduler.c` 文件中实现。   



**调度器初始化函数**   

调度器在使用前需要初始化，具体代码如下：

```c
/* 初始化系统调度器 */
void rt_system_scheduler_init(void)
{	
	register rt_base_t offset;	  // register 修饰变量，防止被编译器优化
	
	/* 线程就绪列表初始化 */
	for (offset = 0; offset < RT_THREAD_PRIORITY_MAX; offset ++)
	{
			rt_list_init(&rt_thread_priority_table[offset]);
	}
	
	/* 初始化当前线程控制块指针 */
	rt_current_thread = RT_NULL;  //(1)
	
	/* 初始化线程休眠列表，当线程创建好没有启动之前会被放入到这个列表 */
	rt_list_init(&rt_thread_defunct);
}
```

(1)处代码，初始化当前线程控制块指针为空。`rt_current_thread` 是在 `scheduler.c` 中定义的一个 `struct rt_thread` 类型的全局指针，用于指向当前正在运行的线程的线程控制块。



**启动调度器**  

调度器启动由函数 `rt_system_scheduler_start()` 来完成，具体实现如下：

```c
/* 启动系统调度器 */
void rt_system_scheduler_start(void)
{
	register struct rt_thread *to_thread;
	

	/* 手动指定第一个运行的线程 */
	to_thread = rt_list_entry(rt_thread_priority_table[0].next,
							  struct rt_thread,
							  tlist);
	rt_current_thread = to_thread;
														
	/* 切换到第一个线程，该函数在context_rvds.S中实现，在rthw.h声明，
       用于实现第一次任务切换。当一个汇编函数在C文件中调用的时候，
       如果有形参，则执行的时候会将形参传人到CPU寄存器r0。*/
	rt_hw_context_switch_to((rt_uint32_t)&to_thread->sp);
}
```

 `rt_list_entry()` 宏前面提到过，作用就是找出线程控制块的起始地址。  



## 第一次线程切换

**rt_hw_context_switch_to() 函数**    

这个函数在 `context_rvds.S` 中实现，用于进行第一次线程切换。   

需要注意的是，当一个汇编函数在 C 语言中被调用时，如果只有一个形参，这个形参会被传入 R0 寄存器，如果有第二个形参，第二形参会被传入 R1 寄存器。  

该函数的实现如下：

```assembly
;*************************************************************************
;                                 全局变量
;*************************************************************************
    IMPORT rt_thread_switch_interrupt_flag
    IMPORT rt_interrupt_from_thread
    IMPORT rt_interrupt_to_thread
		
;*************************************************************************
;                                 常量
;*************************************************************************
;-------------------------------------------------------------------------
;有关内核外设寄存器定义可参考官方文档：STM32F10xxx Cortex-M3 programming manual
;系统控制块外设SCB地址范围：0xE000ED00-0xE000ED3F
;-------------------------------------------------------------------------
SCB_VTOR        EQU     0xE000ED08     ; 向量表偏移寄存器
NVIC_INT_CTRL   EQU     0xE000ED04     ; 中断控制状态寄存器
NVIC_SYSPRI2    EQU     0xE000ED20     ; 系统优先级寄存器(2)
NVIC_PENDSV_PRI EQU     0x00FF0000     ; PendSV 优先级值 (lowest)
NVIC_PENDSVSET  EQU     0x10000000     ; 触发PendSV exception的值
	
;*************************************************************************
;                              代码产生指令
;*************************************************************************

    AREA |.text|, CODE, READONLY, ALIGN=2
    THUMB
    REQUIRE8
    PRESERVE8
		

;/*
; *-----------------------------------------------------------------------
; * 函数原型：void rt_hw_context_switch_to(rt_uint32 to);
; * r0 --> to
; * 该函数用于开启第一次线程切换
; *-----------------------------------------------------------------------
; */
		
rt_hw_context_switch_to    PROC
    
	; 导出rt_hw_context_switch_to，让其具有全局属性，可以在C文件调用
	EXPORT rt_hw_context_switch_to
		
    ; 设置rt_interrupt_to_thread的值
    LDR     r1, =rt_interrupt_to_thread             ;将rt_interrupt_to_thread的地址加载到r1
    STR     r0, [r1]                                ;将r0的值存储到rt_interrupt_to_thread

    ; 设置rt_interrupt_from_thread的值为0，表示启动第一次线程切换
    LDR     r1, =rt_interrupt_from_thread           ;将rt_interrupt_from_thread的地址加载到r1
    MOV     r0, #0x0                                ;配置r0等于0
    STR     r0, [r1]                                ;将r0的值存储到rt_interrupt_from_thread

    ; 设置中断标志位rt_thread_switch_interrupt_flag的值为1
    LDR     r1, =rt_thread_switch_interrupt_flag    ;将rt_thread_switch_interrupt_flag的地址加载到r1
    MOV     r0, #1                                  ;配置r0等于1
    STR     r0, [r1]                                ;将r0的值存储到rt_thread_switch_interrupt_flag

    ; 设置 PendSV 异常的优先级
    LDR     r0, =NVIC_SYSPRI2
    LDR     r1, =NVIC_PENDSV_PRI
    LDR.W   r2, [r0,#0x00]       ; 读
    ORR     r1,r1,r2             ; 改
    STR     r1, [r0]             ; 写

    ; 触发 PendSV 异常 (产生上下文切换)
    LDR     r0, =NVIC_INT_CTRL
    LDR     r1, =NVIC_PENDSVSET
    STR     r1, [r0]

    ; 开中断
    CPSIE   F
    CPSIE   I

    ; 永远不会到达这里
    ENDP

```













**注：文中所有图片均来自《RT-Thread 内核实现与应用开发实战指南》。**
