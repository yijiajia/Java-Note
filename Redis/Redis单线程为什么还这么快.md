

​	Redis之所以快速，一方面是因为它是内存数据库，所有操作都在内存上完成，内存的访问速度本身就很快。另一方面，也要归功于它的数据结构。这是因为键值对是按一定的数据结构来组织的，操作键值对最终就是对数据结构进行增删改查操作。所以高效的数据结构是`Redis`快速处理数据的基础。

​	但是，Redis是单线程的，为什么也能这么快呢？

​	首先通常说的，`Redis`单线程主要是指**Redis的网络IO和键值对读写是由一个线程来完成的，这也是Redis对外提供键值存储服务的主要流程。**但`Redis`的其它功能，例如**持久化、异步删除、集群数据同步**等，其实是由额外的线程执行的。

那问题来了，为什么`Redis`要采用单线程呢？

要明白这问题，关键点在于了解Redis的**单线程设计机制**以及**多路复用机制**。



## 多线程的开销

​	对于一个多线程的系统来说，在有合理的资源分配下，可以增加系统中处理请求操作的资源实体，进而提升系统能够同时处理的请求数，即**吞吐率**。

​	但是因为多线程存在**共享资源并发访问**的问题，即当有多个线程要同时访问修改同个资源时，为了保证资源的正确性，需要采用同步原语来保护共享资源，这样会使得大部分线程处于等待获取锁，系统吞吐率并没有因为线程的增加而增加。同时，这也会降低系统代码的易调试性和可维护性。为了避免这些问题，Redis直接采用了单线程。



Redis单线程快速的原因，一方面是由于前面提到的，底层数据结构和在内存上操作。另一方面，就是Redis采用了**多路复用**机制，使其在网络IO操作中能并发处理大量的客户端请求，实现高吞吐率。





附：Redis 单线程处理IO请求性能瓶颈主要包括两个方面

![image](https://z3.ax1x.com/2021/04/24/cvgAaj.png)



