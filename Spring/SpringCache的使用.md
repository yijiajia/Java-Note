# SpringCache的使用

接口：

* org.springframework.cache.Cache
* org.springframework.cacheManager



在java方法中添加注解，添加注解后会将方法返回的结果保存起来，下一次调用的时候，会将缓存的数据直接返回。



关于缓存存储位置的问题，什么时候该将缓存存储到JVM内部，什么时候该将缓存存储到Redis上面，哪些东西不应该被保存呢。

常久不变的数据且能接受数据的延迟，可以放到JVM缓存中，放到Redis中会有网络的开销；

如果希望集群里面的数据都要一致性的，建议使用分布式缓存（Redis)；

数据经常变化的不应该使用缓存；



**基于注解的缓存**

@EnableCaching

* @Cacheable：主要方法返回值加入缓存。同时在查询时，会先从缓存中取，若不存在才再发起对数据库的访问；
* @CacheEvict：配置于函数上，通常用在删除方法上，用来从缓存中移除对应数据；
* @CachePut：配置于函数上，能够根据参数定义条件进行缓存，与@Cacheable不同的是，每次回真实调用函数，所以主要用于数据新增和修改操作上；
* @Caching：配置于函数上，组合多个Cache注解使用。
* @CacheConfig：主要用于配置该类中会用到的一些共用的缓存配置，配置名字；



原理：使用Aop做增强；





参考链接：

https://blog.csdn.net/zlj1217/article/details/80928122

