## 工作流程
![select-epoll.png](https://github.com/Francis-Tmac/my_notes/blob/main/img/OSandNets/select-epoll.png)

## epoll 函数

#### epoll_create

epoll_create 方法创建了一颗红黑树（epoll 模型），返回epoll 模型句柄

#### epoll_ctl
目的：维护epoll模型（本质是：增删查改红黑树中的节点等）

函数描述：
epoll的事件注册函数，也就是关心哪些文件描述符中的哪些事件（本质就是向红黑树中添加节点以及修改节点还是删除节点）。
不同于select是在监听事件时告诉内核要监听什么类型的事件，在这里epoll是先注册要监听的事件类型。

1. 调用epoll_ctl()向红黑树中添加节点（FD及其感兴趣事件），时间复杂度O(logN)；
2. 向内核的中断处理程序注册一个回调函数，告诉内核如果这个句柄的中断到了，就把它添加到就绪队列中。 所以，当一个socket上有数据到了，内核在把网卡上的数据copy到内核中后就来把socket插入到就绪队列中了；
3. 这个回调函数其实就所把这个事件添加到rdlist这个双向链表中
```$xslt
A、epfd：epoll模型句柄
B、op：表示动作，对于红黑树的增删改。
C、fd：表示需要监听的fd。
D、events：告诉内核要监听什么事。
（C和D表示要关心哪些文件描述符中的哪些事件，红黑树中的节点中的key值就是所关心的文件描述符，而对应的value就是这些文件描述符所关心的哪些事件）。
events可以是下面几个宏的结合：

EPOLLIN : 表示对应的文件描述符可以读
EPOLLOUT : 表示对应的文件描述符可以写。
EPOLLPRI : 表示对应的文件描述符有紧急的数据可读（表示带外数据到来）；
EPOLLERR ： 表示对应的文件描述符发生错误。
EPOLLHUP：表⽰示对应的⽂文件描述符被挂断；
EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于⽔水平触发(LevelTriggered)来说的。
EPOLLONESHOT：只监听⼀一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加⼊入到EPOLL队列⾥里
```

#### epoll_wait
目的：收集所关心的文件描述符上的一些就绪的特定的事件。

调用epoll_wait()返回就绪队列中的就绪事件，时间复杂度O(1)；

ep_poll_callback将相应fd对应epitem加入rdlist，导致rdlist不空，进程被唤醒，epoll_wait得以继续执行
只是检测就绪队列中是否有数据，如果有数据就直接返回，如果没有事件就绪（即该队列为空）时就sleep，直到timeout时间后返回（不论事件是否就绪了）

## select poll epoll 对比

系统调用 | select | poll | epoll
---|---|---|---
事件集合 | '用户通过3个参数分别传入感兴趣的可读，可写及异常等事件内核通过对这些参数的在线修改来反馈其中的就绪事件这使得用户每次调用select都要重置这3个参数' | 统一处理所有事件类型，因此只需要一个事件集参数。用户通过pollfd.events传入感兴趣的事件，内核通过修改pollfd.revents反馈其中就绪的事件 | 内核通过一个事件表直接管理用户感兴趣的所有事件。因此每次调用epoll_wait时，无需反复传入用户感兴趣的事件。epoll_wait系统调用的参数events仅用来反馈就绪的事件
应用程序索引就绪文件描述符的时间复杂度 | O(n) | O(n) | O(1)
最大支持文件描述符数| 一般有最大值限制 |65535 | 65535
工作模式 | LT | LT | 支持ET高效模式
内核实现和工作效率 | 采用轮询方式检测就绪事件，时间复杂度：O(n) | 采用轮询方式检测就绪事件，时间复杂度：O(n) | 采用回调方式检测就绪事件，时间复杂度：O(1)



需要注意的是：epoll并不是在所有的应用场景都会比select和poll高很多。尤其是当活动连接比较多的时候，回调函数被触发得过于频繁的时候，epoll的效率也会受到显著影响！所以，epoll特别适用于连接数量多，但活动连接较少的情况。


参考：

[I/O多路转接之epoll](https://blog.csdn.net/qq_34992845/article/details/76407367)

[Epoll详解](https://blog.csdn.net/yangguosb/article/details/80403432)