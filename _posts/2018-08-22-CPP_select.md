---
layout: post
title:  "多路复用select、poll到epoll"
date:   2018-08-22 09:00:00 +0800
tags: C++ Linux
---

# 为什么要"多路复用"？
为了进行网络通信，我们使用Socket作为连接手段。Linux中一切都是文件，用一个FD(文件描述符)来引用，对网络等IO操作都是类似文件的操作方式，用流的方式读/写。在读/写操作时，就可能发生阻塞，如Socket通信时对方没有发送消息，调用Receive就会阻塞等待，直到收到对方的数据后再继续执行。

当同时连接多个网络IO时（本机是服务器），在单线程下采用阻塞通信就会导致各个通信之间相互阻塞、效率极低。所以我们为每个客户端开启一个线程/进程，专门进行阻塞式通信，这样就可以避免各个连接相互阻塞的情况。

在线程较少的情况下，上面的方法就可以了，但是需求又增加了，多线程/进程的缺点开始显现。当线程数成百上千时，由于IO阻塞交替进行、CPU资源有限等原因需要线程间不断切换。一旦发生线程切换，要涉及大量操作，而多个线程间的操作会是CPU使用效率急剧下降，甚至无法提供服务。

使用单线程异步网络IO的方式就可以避免这种情况。但是为了不断检查各个IO是否可以通信，需要不断循环进行检查，导致CPU大量浪费。

这时，我们只要调用Linux提供的select API，将要等待的阻塞IO的FD写入select集合中，然后就可以进入阻塞状态等待，等到有消息后再处理。这样即避免了多线程的无效切换，又避免了CPU的无效空转。这样的调用方式就叫多路复用(多个通信线路同时重复使用)。

# select
``` C
#include <sys/select.h>
// param: FD集合中最大FD号+1，等待的读set,等待的写set，等待的异常set，超时结构体
int select(int maxfdp, fd_set *readset, fd_set *writeset,
	fd_set *exceptset,struct timeval *timeout);

struct timeval
{      
    long tv_sec;   /*秒 */
    long tv_usec;  /*微秒 */   
};

// 相关宏
int FD_ZERO(fd_set *fdset);		// 初始化FD集合
int FD_SET(int fd, fd_set *fd_set);	// 设置某个要监听的位
int FD_CLR(int fd, fd_set *fdset);	// 清除某个不再监听的位
int FD_ISSET(int fd, fd_set *fdset);	// 测试某个FD是否被置位
```
上面是select涉及的关键函数和宏，调用流程如下：
* 1 初始化`fd_set set; FD_ZERO(&set);`此时set用二进制表示是0000(实际上最大有1024/2048个位)
* 2 设置一个`fd1 = 2,执行FD_SET(fd1,&set);`后set变为0010(第2位置为1)
* 3 设置一个`fd2 = 3,执行FD_SET(fd2,&set);`后set变为0110(第3位置为1)
* 4 上面的fd中最大值是3，所以`fd_max = 4;`
* 5 阻塞执行`res = select(fd_max, &set, 0, 0, 0);`，该函数将set对应数组从用户态拷贝到内核态
	* 5.1 在内核中会先遍历所有fd，检查是否存在IO就绪，如果有就绪的，则保留数组对应标志位，清除未就绪标志位，然后立即返回
	* 5.2 如果没有IO就绪，则进入阻塞状态
	* 5.3 如果设置了超时，而IO一直没有就绪，则到了超时时间也返回
* 6 这时系统收到fd2对应的消息，保留对应的标志位，然后将其它对应的标志位清除，set变为0100，之后将数组拷贝回用户态、返回命中的fd数目
* 7 res被设置为1，然后返回。需要用循环调用`FD_ISSET`检查所有fd，哪些位在数组中仍然存在，则调用相应的处理。(实际使用中，可以通过fd排序、总数计算、最大值记录，减少遍历次数)
* 8 循环进入流程1...

### select特点
* fd数量受到操作系统限制(32位1024/64位2048)，可以监控的IO有限
* 需要经历多次遍历，初始设置每个FD、在内核中检查清除每个FD、返回后需要对每个FD在数组中检查一遍，尤其是返回后的检查时间复杂度是O(n^2)，当并发较大时性能急剧下降
* 需要在用户态和内核态之间频繁传送较大的数组，成本较高
* select中使用的数组数据结构至少是O(n)，随着IO数量增加，性能会持续下降

# poll
poll与select本质上相同，只是数据结构从数组改成了链表，这样就没有了IO数量的限制。

# epoll
epoll在各方面对select和poll的设计进行了改进，将时间复杂度降为O(1)。由于性能出色，很好的解决了C10K问题，被广泛用于高并发服务端设计中，如libevent。

``` C
int epoll_create(int size);		// 创建epoll FD，同时设置要监听的最大数量(注意不是最大FD+1)

struct epoll_event {  
  __uint32_t events;  /* Epoll events */  
  epoll_data_t data;  /* User data variable */  
}; 
/* events的可选Flag
EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）； 
EPOLLOUT：表示对应的文件描述符可以写； 
EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）； 
EPOLLERR：表示对应的文件描述符发生错误； 
EPOLLHUP：表示对应的文件描述符被挂断； 
EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的
EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列 
*/

/*
param: 
	epfd		epoll FD
	op 		操作标志(EPOLL_CTL_ADD添加新的注册、EPOLL_CTL_MOD修改已存在的FD对应的事件、EPOLL_CTL_DEL删除FD)
	fd		注册的FD
	epoll_event	对应的事件
*/
int epoll_ctl(int epfd, int op, int fd,	struct epoll_event *event);

// param: epoll FD，用于返回的events数组，最大event数(不能超过创建epoll时的size)，超时设置(毫秒，0立刻返回)
// return: 返回events的数量，0表示没有，-1表示错误
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

### epoll特点
* 可同时监听的IO数量几乎没有限制，随内存而增加(1G的内存10万)
* 有IO就绪时，只返回就绪的event，不需要遍历，时间复杂度O(1)
* 通过epoll_ctrl函数单独注册FD，并且记录会保存在内核，不需要频繁传送大的数组，提高效率
* epoll底层采用RedBlackTree，时间复杂度为O(logn)，随着IO数量增加会趋于常数，性能较好

### epoll的两种工作模式LT、ET

LT(level-triggered，水平触发，默认工作方式)同时支持block和no-block，内核进行IO就绪通知后，如果不处理，内核还会继续通知。这种模式比较不容易出错，select、poll都是这种工作模式。

ET（edge-triggered，边缘触发）高速模式，只支持non-block，在内核IO就绪通知后，如果不处理，内核不会继续通知，除非某些操作导致那个IO就绪状态变化了。

# epoll与select的选择
并不是在所有场景下，epoll都要比select更好，下面这些场景要权衡使用
* 对于连接总数不大而连接活跃度非常高的服务来说，epoll的性能还不如select
* 相反的，连接数较大而联接不活跃的情况下，epoll比select更好
