# Mybatis 的使用



如何选择Mybatis 和 SpringDataJpa？

> 如果是简单的sql映射，使用jpa即可。

## 使用方式

使用orm框架前，先提前配好数据库的连接参数，使用Java配置 或 properties 文件都可。

### 采用xml

1. 在配置文件中备注mapper.xml文件的位置

   ```yml
   mybatis.mapper-locations=classpath*:/mapper/**/*.xml
   # 使用驼峰转换
   # mybatis.configuration.map-underscore-to-camel-case=true
   ```

2. 创建mapper文件夹，定义xxxMapper接口；
3. 在resources/mapper文件夹下，定义xxxMapper.xml 文件，里面写好方法与sql之间的映射；



### 采用注解

1. 定义xxxMapper 接口，注解为Mapper类（使用@Mapper标记类，表明该接口类为mapper）
2. 定义接口方法，使用 @Insert、@Select、@Delete、@Update表明增删改查。
3. 启动类用@MapperScan注解表名Mapper所在的位置，用于扫描注册mapper。



注解相关的解释及作用：





mapper中的 # 和 ? 的区别



## MyBatis 好用的工具

### 1. MyBatis Generator的使用方式

> 自动生成MyBatis的模板代码









### 2. MyBatis PageHelper的使用方式

> 国人编写的一个工具，支持多种分页方式，多种数据库。

SpringBoot 的支持

```xml
<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper</artifactId>
	<version>最新版本</version>
</dependency>
```

官方文档：https://pagehelper.github.io/docs/



可以使用RowBounds分页、也可以使用pageNum 和 pageSize分页



