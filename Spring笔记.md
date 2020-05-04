# Spring事务的传播机制和隔离级别

## 事务的几种传播特性

1. ​	PROPAGATION_REQUIRED:如果存在一个事务，则支持当前事务，如果没有事务则开启
2. ​    PROPAGATION_SUPPORTS:如果存在一个事务，支持当前事务，如果没有事务，则非事务执行
3. ​    PROPAGATION_MANDATORY:如果存在一个事务，支持当前事务，如果没有一个活动的事务，则抛出异常
4. ​    PROPATATION_REQUIRES_NEW:总是开启一个新的事务，如果一个事务已经存在，则将这个存在的事务挂起
5. ​    PROPAGATION_NOT_SUPPORTED:总是非事务地执行，并挂起任何存在的事务
6. ​    PROPAGATION_NESTED:总是非事务执行，如果存在一个活动事务，则抛出异常
7. ​    PROPAGATION_NESTED:如果一个活动的事务存在，则运行在一个嵌套的事务中，如果没有活动事务，则按TransasctionDefinition.PROPAGATION_REQUIRED属性运行

## Spring事务的隔离级别

1. ISOLATION_DEFAULT: 这是一个PlatfromTransactionManager 默认的隔离级别，使用数据库默认的事务隔离级别，另外四个与JDBC的隔离级别相对应
2. ISOLATION_READ_UNCOMMITTED:这是事务最低的隔离级别，它允许另外一个事务可以看到这个事务未提交的数据，这种隔离级别会产生脏读，不可重复和幻读
3. ISOLATION_READ_COMMITTED(提交读):保证一个事务的修改的数据提交后才能被另外的事务读取，另外一个事务不能读取该事务未提交的数据
4. ISOLATION_REPEATABLE_READ(可重复读):这种事务隔离级别可以防止脏读，不可重读读，但是可能出现幻读，它除了保证一个事务不能读取另一个事务未提交的数据外，还保证了避免下面的情况产生(不可重读读)
5. ISOLATION_SERIALIZABLE(串读): 这是花费最高代价但是最可靠的事务隔离级别，事务被处理为顺序执行，除了防止脏读，不可重复读，还避免了幻读

#### 事务隔离级别：

​	数据库事务的隔离级别有四种：Read_uncommitted,Read_committed,Repeatable_read,Serializable,而且，在事务的并发操作中可能会出现脏读，不可重读读，幻读，事务丢失



- 脏读：（读取了未提交的新事务，然后被回滚了）

​	事务A读取了事务B中尚未提交的数据，如果事务B回滚，则A读取的使用了错误的数据

- 不可重读读：（读取了提交的新事务，指的是更新操作）

​	不可重读是指在对于数据库中的某个数据，一个事务范围内多次查询却返回了不同的数据，但是由于在查询间隔，被另一个事务修改并提交了。

​	幻读（也是读取了提交的新事务，指增删操作）

- 在事务A多次读取过程中，事务B对数据进行了新增操作，导致事务A多次读取的数据不一致

第一类事务丢失（回滚丢失）

- 对于第一类事务丢失，就是比如A和B同时执行一个数据，然后B事务已经提交了，然后A事务回滚，这样B事务的操作就是因A事务回滚丢失的。

- 第二类事务丢失（提交覆盖丢失）
  - 对于第二类事务丢失，也称为覆盖丢失，就是A和B一起执行，两个同时获取到一个数据，然后B事务首先提交，但是A事务接下来又提交，这样就覆盖了B事务
  - 

​	

### SpringBean的生命周期

1. 实例化Bean
2. 设置熟悉值
3. 调用BeanNameAware的setBeanName方法
4. 调用BeanFactoryAware的setBeanFactory方法
5. ApplicatonContextAware的setApplicationContext方法
6. 调用BeanPostProcessor的预初始化方法
7. 调用InitializingBean的afterPropertiesSet方法
8. 调用定制初始化的方法init-method
9. 调用BeanPostProcessor的后初始化方法









AbstractAutowireCapableBeanFacotry.doCreateBean--->



DefaultSingletonBeanRegistry.









BeanDefinitionValueResolver.resolveReference

























































spring 处理循环依赖：

1：AbstractBeanFactory.doGetBean

​	getSingleton(beanName)从单利集合中获取单利对象 最终调用：

​																	defaultSingletonBeanRegistry.getSingleton(String, boolean) 得到单利对象



this.singletonObjects.get(beanName) 从一级缓存中取  取不到则调用beforeSingletonCreation方法 从二级缓存中取还没有查到，然后从三级缓存中singletonFactories 找到对应的对象工厂调用getObject方法，取未完全填充完毕的A的实力对象，然后删除三级缓存数据，填充二级缓存数据，放回对象 





























