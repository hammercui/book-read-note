### 1 make与new的差别，引用类型的意义？
2 逃逸分析？
3 channel的实现？
### 4 gmp与gc?
常见的gc有四种：
1. 引用计数：优点是回收快，实现简单，不需要stw;缺点是无法解决循环引用问题，对cache不友好，使用典型object-c
2. 标记-清除：mark-sweep。古老但有效，如果不可达先标记，到达阈值再stw进行清除。奠定了大部分gc算法的基础。
3. 节点复制 copying: java也用到了，只要是分代回收，不连续分配，在代划分时都会用到copy
4. 分代回收：比如java。



### 5 网络io等待队列？
> io input,output,输入输出，信息交换的流程。
五种io模型：
* 阻塞io(BIO):应用进程向内核发起recfrom请求数据，准备数据报，进程阻塞，从内核复制数据到进程，解除阻塞。
* 非阻塞io(NIO):应用向内核recfrom申请时，缓冲区没有数据，直接返回应用，应用不断尝试。
* io复用模型()：进程将多个fd传递给select，阻塞，等待fd就绪；就绪后通知进程调用recfrom函数，读取数据。核心是一个线程监控多个fd,减少线程分配。
    可同时监控多个fd的函数，就是slect,poll,epoll函数。
    两类线程职能：
    询问线程：select函数监控fd是否有数据
    receiver线程： 询问线程通知去tcp缓冲区拉取数据
* 信号驱动模型：由主动轮询换为被动接受信号，减少大量无效轮询。
* 异步io: 发起read请求，由tcp缓冲区从内核态拷贝入用户态，通知应用线程。异步io天生就是非阻塞的，因为发起read请求后直接就返回了，不会阻塞后续指令。

#### 5.1 在论同步与异步的区别
>内核处理网络数据包有两个阶段，step1 内核空间socket接收缓冲区，这叫数据准备阶段 step2 内核空间拷贝到用户空间，应用读取数据，这叫数据拷贝阶段。

阻塞与非阻塞的区别在于第一阶段，应用询问缓冲区，缓冲区为空时是直接返回还是等待。
而异步与同步的区别在于第二阶段，用户线程执行数据拷贝，成功后返回；还是直接拷贝，完成之后再通知用户线程。

Linux下的 epoll和Mac 下的 kqueue都属于同步 IO。
非阻塞函数：
select: 内核态完成轮询，只能支撑小于1000个并发连接
poll: 改进型的select,只是改进了select只能监听1024个文件描述符的数量限制，但是并没有在性能方面做出改进
epoll： 改进型的poll,epoll只是返回IO活跃的Socket连接。用户进程可以直接进行IO操作。不需要接受socket文件描述符，避免用户空间内核空间的来回复制。
参考：
* [epoll nio区别_一篇文章带你彻底搞懂NIO](https://blog.csdn.net/weixin_39888943/article/details/112014207)
* [通俗理解BIO NIO select epoll并图解举例](https://cloud.tencent.com/developer/article/1773847)
* [bio,nio,aio的区别 select,poll,epoll的区别](https://www.cnblogs.com/eryun/p/12040508.html)
* [深入理解NIO select&epoll](https://zhuanlan.zhihu.com/p/150635981)
* [100%弄明白5种IO模型]([100%弄明白5种IO模型](https://zhuanlan.zhihu.com/p/115912936))
* [5种网络IO模型](https://zhuanlan.zhihu.com/p/54580385)

6 读写屏障
7 map的实现，sync.map的实现，map实现随机的方法

大部分在《go语言设计与实现里》


8 并发actor模型与csp（Communicating Sequential Processes）
>相同点都是通过消息传递来共享内存，并不会产生数据竞争。


## 参考

[Golang 垃圾回收剖析](http://legendtkl.com/2017/04/28/golang-gc/)