1. maven配置时的父模块 与 DeploymentManager节点
2. 





## 常用的JPA 注解

### 实体

* @Entity、@MappedSuperclass
* @Table(name)

### 主键

* @Id
  * @GeneratedValue(strategy,generator)
  * @SequenceGenerator(name,sequenceName)

### 映射

* @Column(name,nullable，length，insertable，updatable)
* @JoinTable(name)、@JoinColumn(name)

### 关系

* @OneToOne、@OneToMany、@ManyToMany、@ManyToOne
* @OrderBy



## Repository

* @EnableJpaRepositories	//自动发现有crud的Repository
* Repository<T,ID> 接口
  * CrudRepository<T,ID>
  * PagingAndSortingRepository<T,ID>
  * JpaRepository<T,ID>



### 根据方法名去定义查询

* find...By.. / read...By... / query..By... /get...By...
* count...By...



### 分页查询

* PagingAndSortingRepository<T,ID>
* Pageable / Sort
* Slice<T> /page<T>



## LomBok常用功能

### 常用注解

* @Getter / @Setter
* @ToString
* @NoArgsConstructor / @RequiredArgsConstructor / @AllArgsConstructor
* @Data (包含@Getter / @Setter  和@ToString)
* @Builder
* @Slf4j / @CommonsLog / @Log4j2





SpringBucks 咖啡项目

利用JPA+Redis；可以考虑使用mongodb做数据库存储；