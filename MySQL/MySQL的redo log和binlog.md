# MySQL的redo log和bin log



## redo log

redo日志即是 **重做日志**，用于存储事务对数据库操作的记录，当数据库发生崩溃重启时，可以通过该日志进行恢复。



​	`MySQL` 中，如果每一次的更新操作都需要写进磁盘，然后磁盘也要找到那条记录，然后再更新，整个过程IO成本、查找成本都很高。`redo`日志的出现正是为了解决这个问题的。



​	执行更新操作时，`MySQL`使用的技术是`WAL`（wirte-Ahead logging）技术（即写前日志），它的关键点在于先写日志，再写磁盘。具体来说，`InnoDB`引擎会先把记录写到`redo log`里面，并更新内存，这个时候更新计算完成了。等适当的时候，一般是空闲时，即会将这个操作记录更新到磁盘里面。



​	redo日志是固定大小的，写入擦除的操作可以理解为是循环链表的操作，存在两个指针`wirte pos `和 `checkpoint`，用于指向文件内存的位置。`wirte pos`是当前记录的位置，一边写一边后移。`checkpoint `是当前要擦除的位置，也是往后推移并且循环的，擦除记录前要把记录更新到数据文件。

<img src="https://z3.ax1x.com/2021/04/17/c4HjBR.png">

​	`wirte pos `和 `checkpoint`中间的部分表示剩余内存， 当`wirte pos `追上 `checkpoint`， 表示redo日志内存已经满了。需要擦除记录后，才能执行新的操作。



​	redo log的存在使得数据库具有`crash-safe`能力，即如果Mysql 进程异常重启了，系统会自动去检查redo log，将未写入到Mysql的数据从redo log恢复到Mysql中去。



### redo log刷盘的时机

​	通常来讲，redo log刷盘的时机是在事务提交的commit阶段采取刷盘的，在此之前，redo log都存在于redo log buffer这块指定的内存区域中。

* log buffer 空间不足时；

  ​	log buffer空间是有限的，通过系统变量innnodb_log_buffer_size大小决定；如果当前写入log buffer的redo 日志量已经占满了50%左右，就需要把这些日志刷新到磁盘中。

* 事务提交时；

  为了保证持久性，必须要把页面修改时所对应的redo日志刷新到磁盘中；否则系统崩溃后，无法将该事务对页面所做的修改恢复过来。

* 后台有一个线程，约每秒一次的频率将log buffer中的redo日志刷新到磁盘；

* 正常关闭服务器时；

* 做chekpoint操作时，即擦除不需要的redo日志时；

## binlog日志

​	`binlog`日志又可以叫**归档日志**，属于Server层的日志，用于存储语句的逻辑（有三种格式）。

它可以用来查看数据库的变更历史、数据库增量备份和恢复、Mysql的复制（主从数据库的复制）。



binlog有三种格式：Statement、Row以及Mixed。

* 基于SQL语句的复制(statement-based replication,SBR)；
* 基于行的复制(row-based replication,RBR)；
* 混合模式复制(mixed-based replication,MBR)。



### binlog 刷盘的时机

​	在事务执行过程中，会先把日志写入到`binlog cache` 中，事务提交的时候，再把`binlog cache`写到 `binlog` 文件中，最后把binlog cache的内容清除掉。

​	写入binlog日志的步骤其实可以分为两步：

1. 先把binlog 从 binlog cache 中写到磁盘上的binlog文件；
2. 调用fsync持久化；



​	一个事务的binlog日志不管有多大都需要确保一次性写入的。每个线程都有自己的binlog cahe，但共同维护一份binlog日志。

​	binlog日志写入是通过变量 `sync_binlog`来控制的，有三个取值情况：

* sync_binlog = 0，表示每次事务提交时，只将日志写入到文件系统的page cache中，写入binlog文件由文件系统决定；
* sync_binlog = 1，表示每次事务提交，都会将数据持久化到磁盘，也即写入binlog文件；
* sync_binlog = N (N>1)，表示每次事务提交，都会写入到文件系统的page cache中，待积累到N个事务后才会写入磁盘；

> 文件系统的page cache 可以理解为文件系统向内核申请的一块内存



​    为保证写入日志的可靠性，以及执行效率问题，通常将值设置成N。设为0的时候，日志丢失的可能性较大；而设置成1时，意味着每次事务提交都要进行一次磁盘写入，这会影响到事务的执行效率。不过设置成N也会个风险，就是主机发生异常重启时，会丢失最近N个事务的binlog日志。建议值在 100 - 1000 中。	



​	使用`show variables like 'sync%';`命令查看binlog写入的配置。这里为1，表示每一个事务提交就写入一次。

[![hBzGBF.png](https://z3.ax1x.com/2021/09/01/hBzGBF.png)](https://imgtu.com/i/hBzGBF)







## 相关问题

### 1. 为什么会有两份日志呢

​	MySQL在5.5之前是没有InnoDB引擎的，默认引擎是`MyISAM`，`MyISAM` 没有`cash-safe`的能力，即**崩溃后重启恢复**的能力，binlog日志只能用于归档。而`InnoDB`是另外一家公司以插件形式引入MySQL的，既然只依靠binlog日志是没有`cash-safe` 能力的，所以`InnoDB`使用了另外一套日志，即redo log日志去实现`cash-safe`能力。



### 2. redo日志和binlog日志的不同

1. redo log是InnoDB引擎特有的日志；binlog是MySQL的Server层实现的，所有引擎都可以使用。
2. redo log是物理日志，记录的是“在某个数据页上做了什么修改”；binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如“给 ID=2 这一行的 c 字段加 1 ”。
3. redo log 是循环写的，空间固定会用完；binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。



### 3. 什么是两阶段提交？

​	两阶段提交是MySQL执行更新操作时，针对relog的写入拆分了两个步骤：`prepare`和`commit`。在存储引擎更新新数据到内存后，会将这个更新记录到`redo log`里面，此时redo log处于`prepare`阶段。然后告知执行器执行完成了，随时可以提交事务。接着执行器会生成这个操作的`binlog`，然后写入磁盘。执行器调用存储引擎事务的提交接口，存储引擎把刚刚写入的`redo log`日志改为`commit`状态，至此更新完成。



注意：当redo log状态为`prepare`时，需要查询对应的binlog事务是否成功，决定回滚还是执行。

<img src="https://z3.ax1x.com/2021/04/17/c4vRPI.png">



**两阶段提交的作用是为了保证redo log 和bin log日志数据的一致性。**

假设redo log 和bin log日志分开写，当第二个日志没有写完期间发生了`crash`。

<img src="https://z3.ax1x.com/2021/04/17/c4xMod.png">