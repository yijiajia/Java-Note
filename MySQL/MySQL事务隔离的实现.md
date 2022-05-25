MySQL 事务隔离的实现依赖于多版本并发控制（MVCC) 和 锁来实现的。

## 多版本并发控制
> 通过在每行记录后面保存两个隐藏的列来实现。一个是行的创建时间、一个是行的过期时间（或删除时间）。存储的并不是实际的时间，而是系统版本号。每新增一个事务，版本号就会自动递增。

在REOETABLE READ 隔离级别下，MVCC具体是如何操作的。

1. SELECT

Innodb 会根据以下两个条件来检查每行记录：

* InnoDB 只查询版本号小于或等于当前事务版本号的数据行，这样可以确保事务读取的行，要么是在事务开始之前就已经存在的，要么就是事务自身插入或修改过的。
* 行的删除版本要么没定义，要么大于当前事务版本号。这可以确保事务读取的当前行在该事务开始之前还没被删除。

满足以上两个条件的记录才会被事务读取返回。

2. INSERT

InnoDB 为新插入的每一行保存当前系统版本号作为行版本号；

3. DELETE

InnoDB 为删除的每一行保存当前系统版本号作为行删除标识；

4. UPDATE

InnoDB 为插入一行新记录，保存当前系统版本号为行版本号，同时保存当前系统版本号到原来的行作为行删除标识。


### 版本链
对于InnoDB引擎来说，在聚簇索引记录中还包含着两个隐藏的列，分别是trx_id 和 roll_pointer。
* trx_id ：当有事务对该记录进行修改时，每次都会将该事务id赋值给trx_id隐藏列。
* roll_pointer：每次对记录修改时，都会把旧版本写入到undo 日志中。这个隐藏列相当于指向undo日志的指针，通过该指针可以找到该记录修改前的信息。

假设表结构为
```
CREATE TABLE `student` (
  `number` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) CHARACTER SET utf8 COLLATE utf8_bin NOT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```
假设表里面已经有name=王五的数据，那么通过下面的事务（事务id=100)去更新该记录，对应的版本链如下图所示。
```
begin 
select * from student where id=1; # 查询得到王五

update student set name='李四' where id=1;

update student set name='张三' where id=1;

commit;
```


[![bx5Kat.png](https://s1.ax1x.com/2022/03/15/bx5Kat.png)](https://imgtu.com/i/bx5Kat)


### ReadView
ReadView，也可以叫做一致性视图。对于读已提交 和 可重复读的事务级别来说，都必须保证读到已经提交的事务修改过的记录。核心问题是，需要判断版本链中的哪个版本是当前事务可见的，为此可以通过ReadView去判断。

ReadView视图的属性值如下：
* m_ids：在生成ReadView时，当前系统中活跃的读写事务的事务id列表。（也即开启了事务但并未提交的事务集合）
* min_trx_id：在生成ReadView时，当前系统中活跃的读写事务中最小的事务id，也即m_ids中最小的值。
* max_trx_id：在生成ReadView时，系统应该分配给下一个事务的事务id值。
> max_trx_id 并不是m_ids数组中的最大值，事务id是递增分配的。假设当前有活跃事务id ：1、2、3，然后3提交了，那么m_ids数组是[1,2],min_tex_id是1，max_trx_id就是4；

* creator_tx_id：生成该ReadView的事务的事务id;
>只有INSERT/DELETE/UPDATE 更新语句才会有事务id，SELECT语句是没有的，也即是默认值0

有了ReadView后，那么在访问某条记录时，可以按照下面的步骤来判断记录的某个版本是否可见：
1. 如果被访问版本的trx_id（事务更新id) 与 ReadView视图中的creator_tx_id相同，表示当前事务在访问它自己修改的记录，所以该版本对于当前事务来说是可见的;
2. 如果被访问版本的trx_id值小于ReadView视图中的min_trx_id 值，表示生成该版本的事务在当前事务生成ReadView视图前就已经提交，所以对于当前事务来说是可见的；
3. 如果被访问版本的trx_id 值西大于或等于 ReadView视图中的max_trx_id 值，表明生成该版本的事务在当前事务生成ReadView后才开启，所以该版本对于当前事务来说不可见；
4. 当trx_id 在ReadView视图的 min_trx_id 和 max_trx 之前，则需要判断trx_id 是否在当前活跃事务组m_ids中；
    * 如果在m_ids数组中，表示创建ReadView时生成该版本的事务还是活跃的，所以该版本不可访问；
    * 如果不在，说明在生成ReadView之前，生成该版本的事务就已经被提交了，所以该版本可见。  

用伪代码可表述如下：
```
/**
* 是否可见
* trx_id ：访问的记录版本的事务更新id
**/
func isVisible(var trx_id) {
    if trx_id == creator_tx_id:
        return true;
    if trx_id < min_trx_id:
        return true;
    if trx_id >= max_trx_id:
        return false;
    if trx_id >= min_trx_id and trx_id < max_trx_id:
        if trx_id in m_ids: 
            return false;
        else:
            return true;
        
}

/**
* 遍历版本链找到当前事务可访问的记录
**/
func getVal() {
    
}

```



READ COMMITED 和 REPEATABLE READ 可以实现不同的隔离级别关键在于两者生成ReadView的时机不同。

#### READ COMMITED 读已提交
> 每次读取数据之前都会生成一个新的ReadView视图，意味着每次使用的ReadView视图都是不一样的，也即需要判断的属性值是不一样的；

假设有个使用READ COMMITED隔离级别的事务开始执行。

[![bx5Kat.png](https://s1.ax1x.com/2022/03/15/bx5Kat.png)](https://imgtu.com/i/bx5Kat)

```
begin;

## SELECT1，此时活跃事务为[100,200]
select * from student where id=1;
```

那么SELECT1 的执行过程如下：
1. 在执行select1 查询语句时，生成一个新的ReadView视图，假设活跃事务m_ids为[100,200]，那么min_trx_id为100，max_trx_id为201，creator_trx_id就为0；
2. 那么遍历版本链，拿到最新版本的trx_id；由上图可知最新版本的trx_id为100，在m_id是数组中，那么说明对当前事务不可见；
3. 拿到下一个"李四" 的记录，trx_id 也为 100，同意不符合条件，继续获取下一个；
4. 拿到"王五"记录的trx_id=80，小于 min_trx_id，说明当前版本在事务开始之前就已经提交，也即是可见的；


假设此时事务100提交了，事务100的执行语句如下：
```
begin;

update student set name='李四' where id=1;

update student set name='王五' where id=1; 

commit;
```

事务200的执行如下：
```
begin;

update student set name='张飞' where id=1;

update student set name='关羽' where id=1; 
```

那么此时的版本链为：
[![bx7GYq.png](https://s1.ax1x.com/2022/03/15/bx7GYq.png)](https://imgtu.com/i/bx7GYq)


那么在前面的查询事务中，再次查询该记录
```
begin;

## SELECT1，此时活跃事务为[100,200]
select * from student where id=1;

## SELECT2，此时事务100已提交
select * from student where id=1;
```

那么SELECT2 查询的分析如下：
1. 在执行SELECT2 语句时又会单独生成一个ReadView视图。此时ReadView视图中m_ids列表为[200]，mix_trx_id为200，max_trx_id为201，creator_trx_id 为0；
> 事务100已经提交了，所以新视图中没有该事务，这也是跟可重复读实现结果不同的原因
2. 遍历版本链，发现最新记录的trx_id=200，在m_ids数组中，那么对于当前事务来说是不可见的；
3. 拿到下一个"张飞"的记录，trx_id同样为200，与上面一致，不可见；
4. 拿到下一个"张三"记录，trx_id为100，小于当前ReadView的min_trx_id，那么当前版本对于该查询来说是可见的，返回该记录；


可以发现，在读已提交的隔离级别下，在同一个事务中，两次查询的结果是可能不一样的，第二次的查询结果就是另一个事务100刚提交的数据；




#### REPEATABLE READ 可重复读
> 只在第一次读取数据的时候才会创建新的ReadView视图，意味着在事务执行期间，使用的ReadView视图都是同一个不变的。


对于前面对读已提交隔离级别的分析来看，如果是可重复读的隔离级别，那么这两次的查询结果都是一样的，也即都是王五这条记录。因为在这两次查询中，事务生成的ReadView视图只有一个。这也就是可重复读的定义。


## 锁

 

## 小结
从前面的分析中可以看出，所谓的MVCC其实就是在使用READ COMMITED、REPEATABLE READ 这两种隔离级别的事务执行普通的SELECT操作时，访问版本链的过程。这样可以使不同的事务的读-写、写-读操作并发执行，从而提升系统性能。


## 文档 

隐式自动提交的命令：https://dev.mysql.com/doc/refman/5.6/en/implicit-commit.html