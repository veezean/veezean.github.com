---
layout: post
title: "iBatis & myBatis & Hibernate 要点记录"
date: 2017-09-29
categories: SQL
tags: JAVA iBaitis myBatis Hibernate
---

* content
{:toc}


这三个是当前常用三大持久层框架，对其各自要点简要记录，并对其异同点进行简单比较。




## 1. iBatis

iBatis主要完成两件事情：
> 1. 根据JDBC规范建立与数据库之间的连接、并进行连接管理与事务管理；
> 2. 通过反射打通JAVA对象与数据库参数交互之间的相互转换关系。

基本使用步骤：
> 1. 在xml配置文件中配置上数据库的连接池信息、事务相关的参数等，并指定要加载的SqlMapConfig文件；
> 2. 在数据表对应的SqlMap xml配置文件中，定义数据表字段与JAVA Bean对象字段之间的映射关系，并定义需要执行的相关操作，比如select、update、delete、insert等操作对应的SQL语句；
> 3. 在SqlMapConfig xml配置文件中，配置上步骤2中提及的每个数据表对应的配置映射文件；
> 4. JAVA中定义好对应的JavaBean对象；
> 5. 在JAVA中实现DAO层的代码逻辑，封装对应的增删改查接口提供给业务层使用。在DAO层的具体方法实现中，分别调用SQLMapClient实例中提供的增删改查方法，传入此前在步骤3中的xml映射文件中定义的SQLID，以及相关必要的参数。

特点：
> 1. ibatis以SQL为中心的持久化层框架，其可以将SQL执行的结果映射到JAVA类对象上，同时也将JAVA对象映射为SQL的输入参数进行执行。
> 2. ibatis简单易学，相对而言灵活性非常的高，在SQLMap中的SQL语句可以支持较为复杂个性化定制的SQL语句，当SQL执行存在性能瓶颈的时候，iBatis可以直接通过优化xml中的SQL语句来提升性能，具有更好的可控性。
> 3. iBatis 封装了绝大多数的 JDBC 样板代码，使得开发者只需关注 SQL 本身，而不需要花费精力去处理例如注册驱动，创建 Connection，以及确保关闭 Connection 这样繁杂的代码。


## 2. myBatis

myBatis 作为 iBatis 高版本（iBatis只到2.x版本， 随后开发团队转投Google旗下，iBatis 3.x版本正式更名mybatis），主要是相对于iBatis 在细节层面会有所增强，配置起来更加简洁、在对象关系映射方面有所改进，效率有所提升。


## 3. hibernate

优点：
> 1. DAO层的开发比较简单，不像myBatis还需要自行维护SQL与结果映射；
> 2. 对象维护与缓存机制，比myBatis要好，对增删改查的对象维护要更加方便；
> 3. 由于不需要写SQL语句，因此数据库的移植性会更好；

缺点：
> 1. 不像myBatis，没有办法进行更为细致的SQL优化，无法减少无用查询字段；
> 2. 入门门槛相对较高些。



## 4.异同点比较

> 1. iBatis或者myBatis都需要手动编写SQL语句，hibernate不需要关注SQL的生成与结果的映射，可以更加专注于业务流程的实现；
> 2. iBatis、myBatis相对于hibernate更加容易上手，且可以进行更为细致的SQL优化，减少查询字段；
> 3. 

缓存机制差别比较：
> 1. 两者的二级缓存，除了系统默认之外，还可以完全自定义；
> 2. 由于hibernate用户不需要关注SQL，因此不用担心脏数据问题，因为如果出现脏数据的话，框架会报错；但是使用myBatis的二级缓存的时候需要小心，避免出现脏数据影响系统正常使用。

选择要点：

**使用 Hibernate 的开发者应该总是关注对象的状态（state），不必考虑 SQL 语句的执行。**

> 1. 根据**使用场景**来决定，如果一个项目组基本都是最简单的增删改查，则hibernate就很快，因为基本的SQL都已经封装好了，开发过程中不需要再去写SQL语句了，可以节省大量时间。但是如果项目中需要使用较多的复杂语句，或者对性能要求极其高，需要对SQL进行更为细致化的优化的时候，选择myBatis会更加好一些。