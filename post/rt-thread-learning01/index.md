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



# 线程与线程的切换

前面讲到，在裸机系统中，系统的主体就是 main 函数里面顺序执行的无限循环，这个无限循环里面 CPU 按照顺序完成各种事情。在多线程系统中，我们根据功能的不同，把整个系统分割成一个个独立的且无法返回的函数，这个函数我们称为线程。

## 线程的创建

### 定义线程栈

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

### 定义线程函数

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

### 定义线程控制块

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



### 线程创建函数的实现

创建线程就是根据给出的参数如线程的名称、线程栈的大小等完成线程栈的初始化和线程控制块的初始化。线程的创建由函数 `rt_thread_init()` 来实现，该函数在 `thread.c` 中定义在 `rthread.h` 中声明。该函数的实现如下：

```c
rt_err_t rt_thread_init(struct rt_thread *thread,        // 线程控制块
                        void (*entry)(void *parameter),  // 线程函数名即线程入口
                        void             *parameter,     // 线程形参
                        void             *stack_start,   // 线程栈起始地址
                        rt_uint32_t       stack_size)    // 线程栈大小
{
	rt_list_init(&(thread->tlist));                      // (1)
	
	thread->entry = (void *)entry;
	thread->parameter = parameter;

	thread->stack_addr = stack_start;
	thread->stack_size = stack_size;
	
	/* 初始化线程栈，并返回线程栈指针 */
	thread->sp = (void *)rt_hw_stack_init( thread->entry, 
		                                   thread->parameter,
							               (void *)((char *)thread->stack_addr + thread->stack_size - 4) );
	
	return RT_EOK;
}
```

标号为 (1) 的代码对线程控制块的线程链表节点 `tlist` 进行初始化，链表节点可以插入到不同的链表，线程调度器通过链表调度线程。`tlist` 的数据原型是 `rt_list_t` 。   

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



```c
/* 在双向链表头部插入一个节点 */
rt_inline void rt_list_insert_after(rt_list_t *l, rt_list_t *n)
{
    l->next->prev = n; /* 第1步 */
    n->next = l->next; /* 第2步 */
    
    l->next = n; /* 第3步 */
    n->prev = l; /* 第4步 */
}
```



{{% figure class="center" src="/img/rt-thread-learning01/list_op_2.jpg" alt="在双向链表表头后面插入一个节点" title="在双向链表表头后面插入一个节点" %}}   



**在双向链表表头前面插入一个节点**   



```c
rt_inline void rt_list_insert_before(rt_list_t *l, rt_list_t *n)
{
    l->prev->next = n; /* 第1步 */
    n->prev = l->prev; /* 第2步 */
    
    l->prev = n; /* 第3步 */
    n->next = l; /* 第4步 */
}
```

{{% figure class="center" src="/img/rt-thread-learning01/list_op_3.jpg" alt="在双向链表表头前面插入一个节点" title="在双向链表表头前面插入一个节点" %}}      



**从双向链表删除一个节点**   

```c
rt_inline void rt_list_remove(rt_list_t *n)
{
    n->next->prev = n->prev; /* 第1步 */
    n->prev->next = n->next; /* 第2步 */
    
    n->next = n->prev = n; /* 第3步 */
}
```

{{% figure class="center" src="/img/rt-thread-learning01/list_op_4.jpg" alt="从双向链表删除一个节点" title="从双向链表删除一个节点" %}}      





**注：文中所有图片均来自《RT-Thread 内核实现与应用开发实战指南》。**
