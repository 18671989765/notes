# 自定义持久层框架MyBatis笔记

## 1：MyBatis动态sql

```java
	<select  id = "findByIds"  parameterType="list" resultType="user">
	    select * from User
        	<where>
        		<foreach collection="array" open="id in(" close=")" item="id" separator=",">
          			#{id}      	
                </foreach>
        	</where>
	</select>
```
- foreach: 标签的属性含义：
- ​	标签遍历集合的属性：
- ​			collection: 代表要遍历的集合元素，注意编写是不要写#{}
- ​			open： 代表语句的开始部分
- ​			close：代表结束部分
- ​			item：代表遍历集合的每个元素，生成的变量名
- ​			sperator: 代表分隔符



## MyBatis复杂映射开发

### 一对一查询：

​		用户表和订单表的关系：一个用户有多个订单，一个订单只从属于一个用户

​		一对一查询需求：查询一个订单与此同时查询出该定单所属的用户信息



​		

```java
<resultMap id="orderMap" type="com.lagou.domain.Order">

​				<result property="id" column="id"></result>

​				<result property="ordertime" column="ordertime"></result>

​				<association property="user"  javaType="com.lagou.domain.User>

​						<result property="uid" property="id"/>

​						<result property="username" property="username"/>

​				</association>`

​		</resultMap>
```



### 一对多查询模型

​		用户表和订单表的关系为：一个用户有多个订单，一个订单属于一个用户

​		一对多查询的需求：查询所有用户，与此同时查询该用户具有的订单

​		select * ,o.id oid from user u left join orders on u.id=o.uid;



​		

```
<resultMap id = "userMap" type="com.lagou.domain.User">
	<result column="id" property="id"/>
	<result column="username" property="username"/>
	<collection property="orderList" ofType="com.lagou.domain.Order">
		<result column="oid" property="id"/>
		<result column="ordertime" property="ordertime"/>
	</collection>
	
</resultMap>

<select id="findAll" resultMap="userMap">
	select * ,o.id,oid from user u left join orders o on u.id = o.uid
</select>
```

### 多对多查询

多对多查询模型

用户表和角色表的关系为：一个用户有多个角色，一个角色被多个用户使用

多对多查询需求：查询用户同时查询该用户所有角色



```
<resultMap id = "userRoleMap" type="com.lagou.domain.User">
	<result column = "id" property = "username"/>
	<result column = "username" property="username"/>
	<collection property="roleList"  ofType="com.lagou.domain.Role">
		<result column="rid" property="id"/>
		<result column = "rolename" property="rolename"/>
	</collection>
	
	<select id = "findAllUserAndRole" resultMap="userRoleMap">
		select u.*,r.*,rid from user u left join user_role ur on u.id = ur.user_id
		inner join role r on ur.role_id = r.id
	</select>
</resultMap>
```



## MyBatis注解开发：

1. @insert 实现新增
2. @update 更新
3. @delete 删除
4. @select 查询
5. @Result 结果集封装
6. @Results 可以与@Result 一起用 封装多个结果集
7. @one 一对一查询结果集封装
8. @Many 一对多结果集封装





### MyBatis缓存

- 缓存就是内存中的数据，常常来自对数据库查询结果的保存，使用缓存，我们可以避免频繁与数据库交互，进而提高响应速度
- SqlSession 属于一级缓存
- Mapper(namespace)二级缓存
- （1）一级缓存是SqlSession 级别的缓存，在操作数据库时需要构造sqlSession对象，在对象中有一个数据结构 （HashMap） 用于缓存数据，不同的sqlSession之间的缓存数据区域是相互不影响的
- （2）二级缓存是mapper级别的缓存多个sqlSession去操作同一个Mapper的sql语句，多个sqlSession可以共用二级缓存，二级缓存是跨sqlSession的



##### 我们在一个sqlSession中对User表根据id进行两次查询，查看他们发出的sql语句情况

 

```
SqlSession sqlsession = sessionFactory.openSession();
UserMapper userMapper = sqlsession.getMapper(UserMapper.class);
//第一次查询发出sql语句，并将查询结果放入缓存中
User u1 = userMapper.selectUserByUserId(1);
System.out.println(u1);

//第二次查询，由于是同一个sqlSession会在缓存中查询结果
User u2 = userMapper.selectUserByUserId(1);
System.out.println(u2);

sqlSession.close();


=====如果是两次update操作，则都会走数据库查询

总结：
	1：第一次发起查询用户id为1的用户信息先去找缓存中是否有id为1的用户信息，如果没有，从数据库查询用户信息，得到用户信息，将用户信息存储到一级缓存
	2：如果中间sqlsession去执行commit操作（执行插入，更新，删除） 则会清空sqlSession中的一级缓存，这样做的目的为了让缓存中的是最小的数据，避免脏度
	3：第二次发起查询用户id为1的用户信息，先去找缓存中是否是id为1的用户信息，缓存中有，直接从缓存中获取用户信息
```



二级缓存整合redis

```
mybatis提供了一个cache接口，如果要实现自己的缓存逻辑，实现cache接口开发即可，mybatis本身默认实现了一个，但是这个缓存实现无法实现分布式缓存，所以我们这里使用redis分布式就可以，mybatis提供了一个针对cache接口的reids实现类，该类存在mybatis-redis包中
：pom文件

	<dependency>
		<groupId>org.mybatis.caches</groupId>
		<artifactId>mybatis-redis</artifactId>
		<version>1.0.0-beta2</version>
	</dependency>
	
	
```



#### Mybatis插件介绍：

​	
$$
Mybatis是一个优秀的ORM开源框架，这个框架具有很强大的灵活性，在四大组件(Executor,StatementHandler,ParameterHandler,ResultSetHandler)提供了简单易用的插件扩展机制，mybatis对持久层的操作就是借助于四大核心对象功能，增强功能本质上是借助底层动态代理实现
$$


##### MyBatis运行拦截的方法：

1. ​		执行器Executor(update,query,commit,rollback等方法)
2. ​		Sql语法构建器StatementHandler(prepare,parameterize,batch,update,query)
3. ​		参数处理器：ParameterHandler(getParameterObject,setParameters方法)
4. ​		 结果集处理器ResultSetHandler  handlerResultSets,handlerOutputParameters



##### 	pageHelper分页插件：





##### 通用mapper：

​	通用mapper就是解决单表增删改查，基于mybatis的插件机制，开发人员不需要写sql，不需要再DAO中增加方法，只要写好实体类，就能支持相应的增删改查方法



1：在maven项目，pom.xml中引入mapper依赖

​		<dependency>

​			<groupId>tk.mybatis</groupId>

​			<artifactId>mapper</artifactId>

​			<version>3.1.2</version>

</dependency>



​		

​		





