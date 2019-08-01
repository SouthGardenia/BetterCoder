# SpringBoot的数据访问
&emsp;Spring Data项目就是用来解决数据访问问题的，其包含了大量关系型数据库及非关系型数据库的数据访问解决方案。
## 声明式事务
### Spring的事务机制
&emsp;所有的数据访问技术都有事务处理机制，这些技术提供了API用来开启事务、提交事务来完成数据操作，或者在发生错误时回滚数据。而Spring的事务机制是用统一的机制来处理不同数据访问技术的事务处理，Spring的事务机制提供了PlatformTransationManager接口，不同的数据访问技术的事务使用不同的接口实现。
| 数据访问技术 | 实现|
|--|--|
| JDBC|DataSourceTransationManager|
| JPA|JpaTransactionManager|
| Hibernate|HibernateTransactionManager|
| JDO|JdoTransactionManager|
### 声明式事务
用@EnableTransactionManagement注解在配置类上，开启声明式事务的支持，@Transactional注解用来表示开启事务支持

### 注解事务行为
1.  事务传播特性:propagation

|  属性| 含义|
|--|--|
| REQUIRED| 默认值。方法A调用时没有事务则新建一个事务，A调用方法B时，B将使用和A相同的事务|
| REQUIRED_NEW| 方法A和方法B，在方法调用的时候都将开启一个新的事务|
| NESTED| 和REQUIRED_NEW相似，只支持JDBC|
| SUPPORTS| 方法调用时有事务就用事务，没有事务则不使用事务|
| NOT_SUPPORTS| 不使用事务，若有事务则被挂起至结束|
| NEVER| 不使用事务，有事务则抛异常|
| MANDATORY| 强制在事务中执行，若无事务抛出异常|
1.  事务隔离级别：isolation

| 属性| 含义|
|-- | --|
| READ_UNCOMMITED| A事务修改了一条数据但没提交事务，B可以读到修改后的记录，会导致脏读、不可重复读、幻读|
| READ_COMMITED| A修改了一条数据提交了事务，B事务才可以读到修改后的记录，解决了脏读|
| REPEATABLE_READ| 当A读取了一条记录，B事务将不允许改这条记录，解决了额不可重复读|
| SERIALIZABLE| 串行执行，可以解决上述问题|
| DEFAULT| 使用数据库的默认隔离级别|

## 数据缓存Cache
### Spring的缓存支持