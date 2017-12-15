---
title: Spring 事务
date: 2017-4-26
tags: Spring
categories: Spring
---

# 事务

1. 事务：逻辑上的一组操作,这组操作要么全部成功,要么全部失败
2. 事务四大特性
	- **原子性**: 事务是一个不可分割的工作单位,事务中的操作要么都发生,要么都不发生
	- **一致性**: 事务前后数据的完整性必须保持一致(例如:两个人转账,转账前后总金额的数目都是固定的)
	- **隔离性**: 多个用户并发访问数据库时,一个用户的事务不能被其他用户的事务所干扰,多个并发事务之间数据要相互隔离(例如:假设有两个事务同时在操作数据库,例如张三修改一个记录,同时李四也在修改这个记录,会导致该记录被重复修改,或者第一次修改的记录被第二次记录给覆盖掉)
	- **持久性**: 一个事务一旦被提交,它对数据库中数据的改变就是永久性的,即使数据库发生故障也不应该对其有任何影响

<!-- more -->

# 事务隔离

通常来说，在一个应用中，会同时有多个用户访问数据库，在并发访问数据库的时候，可能会有，以下三个问题
1. **脏读**
	一个事务读取了另一个事务改写但还未提交的数据，如果这些数据被回滚，则读到的数据是无效的。
2. **不可重复读**
	在同一个事务中，多次读取同一数据返回的结果有所不同。
3. **幻读**(“幻读”是不可重复读的一种特殊场景)
	一个事务读取了几行记录后，另一个事务插入一些记录，幻读就发生了。再后来的查询中，第一个事务就会发现有些原来没有的记录。


**DBMS通过加锁来进行并发控制，不同的隔离级别拥有不同的持锁时间。**

"C"-表示锁会持续到事务提交。 "S" –表示锁持续到当前语句执行完毕。如果锁在语句执行完毕就释放则另外一个事务就可以在这个事务提交前修改锁定的数据，从而造成混乱。

|隔离级别l	|写操作	|读操作	|范围操作 (...where...)
|---     |---   |---       |---  
|未提交读	 |S	   |S	        |S
|提交读	  |C	  |S	       |S
|可重复读	  |C	 |C	        |S
|可序列化	  |C	 |C	        |C

**隔离级别**
1. 未提交读(`READ_UNCOMMITED`);
允许你读取还未提交的改变了的数据，可能导致脏，幻，不可重复读。
2. 提交读(`READ_COMMINTED`):
允许在并发事务已经提交后读取，可防止脏读，但幻读和不可重复读还是有可能发生。
3. 可重复读(`REPEATABLE_READ`):
对相同字段的多次读取是一致的，除非数据被事务本身改变，可防止脏读，不可重复读，但幻读仍有可能出现。
4. 可序列化(`SERILIZABLE`):
完全服从ACID的隔离级别，确保不发生脏读，幻读，不可重复读，这在所有的隔离级别中是最慢的，它是典型的完全通过锁定在事务中涉及的数据表来完成的。

如下表所示：

|隔离级别	|脏读   |	不可重复读|	幻影读|
|---     |---   |---       |---   |
|未提交读	|可能发生|可能发生  |	可能发生
|提交读	 |-	    |可能发生	 |可能发生
|可重复读	|-	    |-	     |可能发生
|可序列化	| -	    | -	      |-


TransactionDefinition定义事务隔离级别

**默认事务隔离级别**

除了以上的数据库提供的事务隔离级别，spring提供了Default隔离级别，该级别表示spring使用后端数据库默认的隔离级别。
- MySQL：REPATABLE_READ(可能出现幻读)
- Oracle：READ_COMMITTED(可能出现不可重复读和幻读)


**并发操作怎么设置数据库隔离级别**

这个要看你具体使用场景，例如你是并发的时候去写，还是去读，还是去读写，一般来说是读写，那么一般是 repeatable read，虽然他有幻读得可能性，但是一般去发生幻读得业务是不常见的。

# 事务传播行为
事务的传播行为：解决业务层的方法之间的相互调用的问题(在调用方法的过程中,事务是如何传递的)


事务的传播行为有七种，又分为三类：
1. 如果 A 方法中有事务，则调用 B 方法时就用该事务，即：A和B方法在同一个事务中。
	- **PROPAGATION_REQUIRED**：如果 A 方法中没有事务，则调用 B 方法时就创建一个新的事务，即：A和B方法在同一个事务中。
	- **PROPAGATION_SUPPORTS**：如果 A 方法中没有事务，则调用 B 方法时就不使用该事务。
	- **PROPAGATION_MANDATORY**：如果 A 方法中没有事务，则调用 B 方法时就抛出异常。
2. A 方法和 B 方法不在同一个事务里面。
	- **PROPAGATION_REQUIRES_NEW**：如果 A 方法中有事务，则挂起并新建一个事务给 B 方法。
	- **PROPAGATION_NOT_SUPPORTED**：如果 A 方法中有事务，则挂起。
	- **PROPAGATION_NEVER**：如果 A 方法中有事务，则报异常。
3. 如果 A 方法有的事务执行完，设置一个保存点，如果 B 方法中事务执行失败，可以滚回保存点或初始状态。
	- **PROPAGATION_NESTED** ：如果当前事务存在，则嵌套事务执行

重点的三种：

- PROPAGATION_REQUIRED
- PROPAGATION_REQUIRES_NEW
- PROPAGATION_NESTED

# Spring事务管理

Spring将事务管理分成了两类:
* 编程式事务管理
	- 手动编写代码进行事务管理(很少使用)
* 声明式事务管理
	- 基于TransactionProxyFactoryBean的方式(很少使用),需要为每个进行事务管理的类,配置一个TransactionProxyFactoryBean进行增强
	- 基于AspectJ的xml方式(经常使用),一旦配置好,类上不需要添加任何东西
	- 基于注解(经常使用),配置简单,需要在业务层类上添加一个@Transactionl的注解


# 编程式事务管理

## TransactionProxyFactoryBean

```xml
<bean id="accountServiceProxy" class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">

        <property name="target" ref="accountService"/>

        <property name="transactionManager" ref="transactionManager"/>

        <property name="transactionAttributes">
            <props>
                <!--
                   prop格式：
                       * PROPAGATION：事务的传播行为
                       * ISOLATION	：事务的隔离级别
                       * readOnly	：只读(不可以进行修改,插入,删除的操作)
                       * -Exception	：发生哪些异常回滚事务
                       * +Exception	：发生哪些异常事务不回滚
                 -->
                <prop key="transfer">PROPAGATION_REQUIRED,readOnly</prop>
            </props>
        </property>
    </bean>
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
```
## AspectJ

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 配置事务的通知（事务的增强） -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <!--
                propagation	：传播行为
                isolation	：隔离级别
                read-only	：只读
                rollback-for	：发生哪些异常时回滚
                no-rollback-for	：发生哪些异常时不回滚
                timeout	：过期信息
             -->
            <tx:method name="transfer" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>
    <!-- 配置aop切面 -->
    <aop:config>
        <!-- 配置切入点 -->
        <aop:pointcut expression="execution(* net.morethink.service.AccountService+.*(..))" id="pointcut1"/>
        <!-- 配置切面 -->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut1"/>
    </aop:config>
```
## @Transactionl

```xml
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <tx:annotation-driven transaction-manager="transactionManager"/>
```

参考文档
1. [事务隔离-维基百科](https://zh.wikipedia.org/wiki/%E4%BA%8B%E5%8B%99%E9%9A%94%E9%9B%A2)
2. [全面分析 Spring 的编程式事务管理及声明式事务管理-IBM](https://www.ibm.com/developerworks/cn/education/opensource/os-cn-spring-trans/)
3. [MyBatis6：MyBatis集成Spring事物管理（下篇）-博客园](http://www.cnblogs.com/xrq730/p/5454381.html)
