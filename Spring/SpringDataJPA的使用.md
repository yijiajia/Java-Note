# SpringData JPA 的使用

## 简介

>SpringDataJpa 是SpringData项目中的一个子项目。主要是针对物理数据层的抽象做了实现。通过对象映射，实现域对象和持续化存储之间的转换。通过实现特定的接口类，即可在无需编写SQL 以及 简单的查询语句 都能做到对数据库的访问。
>
>此外，SpringData的项目还包括SpringDataRedis 和 SpringData MongoDB这样的NoSQL 实现。



通过IDEA 查看到SpringDataJpa 中的接口实现结构图。

![image-20210116135516545](C:\Users\45850\Desktop\MD笔记图\SpringDataJPA的使用\image-20210116135516545.png)



## 常用的JPA 注解

### 1. 实体

* @Entity、@MappedSuperclass

* @Table(name)

  > 注解了@Entity的Model对象会映射成数据表；
  >
  > @MappedSuperclass 表示这个bean对象是mapper映射的父类，一般是用于多个bean对象有相同属性时，而抽离出来做父类的bean;
  >
  > @Table 注解用于映射Entity对象对应的表名，默认表名跟Entity类名一致。

### 2. 属性

* 标明主键 @Id

  * @GeneratedValue(strategy,generator)
  * @SequenceGenerator(name,sequenceName)

  > @GeneratedValue 用于标注主键的生成策略，通过strategy 属性指定。默认情况下，JPA 自动选择一个最适合底层数据库的主键生成策略：SqlServer对应identity，MySQL 对应 auto increment。 
  > 在javax.persistence.GenerationType中定义了以下几种可供选择的策略： 
  > –IDENTITY：采用数据库ID自增长的方式来自增主键字段，Oracle 不支持这种方式； 
  > –AUTO： JPA自动选择合适的策略，是默认选项； 
  > –SEQUENCE：通过序列产生主键，通过@SequenceGenerator 注解指定序列名，MySql不支持这种方式 
  > –TABLE：通过表产生主键，框架借由表模拟序列产生主键，使用该策略可以使应用更易于数据库移植。3. 映射

* @Column(name,nullable，length，insertable，updatable)

> name：字段名
>
> nullable：表示该字段是否可以为null值，默认为true
>
> length：字段长度，当字段的类型为varchar时，该属性才有效，默认为255个字符
>
> insertable：表示在使用“INSERT”脚本插入数据时，是否需要插入该字段的值
>
> updatable：表示在使用“UPDATE”脚本插入数据时，是否需要更新该字段的值。一般用于只读属性，例如主键、外键等。

* @JoinTable(name)、@JoinColumn(name)

> 

### 4. 关系

* @OneToOne、@OneToMany、@ManyToMany、@ManyToOne
* @OrderBy



## 常用的Repository接口

* @EnableJpaRepositories	//自动发现有crud的Repository
* Repository<T,ID> 接口
  * CrudRepository<T,ID>
  * PagingAndSortingRepository<T,ID>
  * JpaRepository<T,ID>



### 1. 根据方法名去定义查询

* find...By.. / read...By... / query..By... /get...By...
* count...By...



### 2. 分页查询

* PagingAndSortingRepository<T,ID>
* Pageable / Sort
* Slice<T> /page<T>





