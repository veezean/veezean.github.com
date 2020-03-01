---
layout: post
title: "SpringBoot中@Transaction在不同MySQL引擎下的差异性"
date: 2019-07-16
categories: SpringBoot
tags: JAVA Spring SpringBoot 事务 MySQL MyISAM InnoDB
---

在springboot进行事务测试的时候，发现事务没有生效，记录一下过程。





## SpringBoot中@Transaction在不同MySQL引擎下的差异性

在springboot进行事务测试的时候，发现事务没有生效，在方法上添加了`@Transactional`注解并让方法先执行插入操作，接着再抛出个异常，触发事务回滚，代码如下：

```java
@Transactional(propagation = Propagation.REQUIRED, rollbackFor =Exception.class)
public void executeSaveRollback(TestTable testTable) throwsException {
    repository.saveAndFlush(testTable);
    // 抛出异常，触发事务回滚
    throw new Exception("test throw exception and rollback the transaction...");
}
```
实际测试的时候发现，事务回滚并没有效果，数据已经插入到DB中：

![](/assets/post_pics/2019-07-16-springboot%20transaction.md/md_pics_2019-07-16-19-25-47.png)

搜索了下，发现有人提到说MySQL数据库的事务不生效，可能和引擎类型有关系，因此下一步往这个方向排查一下。


### 查看MySQL数据表的引擎类型

通过`show create table TABLE_NAME`命令查看MySQL中数据表对应的引擎类型，查询结果如下所示：

```sql
Table	Create Table
TestTable	CREATE TABLE `TestTable` (
  `uniqueId` bigint(20) NOT NULL,
  `objectDesc` varchar(255) COLLATE utf8mb4_general_ci DEFAULT NULL,
  `objectName` varchar(255) COLLATE utf8mb4_general_ci NOT NULL,
  `CreateTime` datetime NOT NULL,
  `UpdateTime` datetime DEFAULT NULL,
  PRIMARY KEY (`uniqueId`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci
```

从上面的内容中可以看到`ENGINE=MyISAM`即表示了使用的是MyISAM引擎，所以测试代码中的事务回滚没有生效。

### 修改JPA自动创建表时的引擎

测试工程中，使用的是JPA自动创建数据表的方式，默认情况下，创建出来数据表使用的是MyISAM引擎，因此如果需要使用事务，可以手动配置下，指定使用InnoDB引擎进行创建数据表。

在SpringBoot的application.properties配置文件中，加入如下一行配置即可：

> spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect

完整的JPA相关的配置参数如下：

```properties
# JPA Configure
# database type
spring.jpa.database=mysql
# whether to show the sql in the log or console
spring.jpa.show-sql=true
spring.jpa.hibernate.naming.implicit-strategy=org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyJpaImpl
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect
```
重新启动测试进程，然后再次执行`show create table TABLE_NAME`结果如下：

```sql
CREATE TABLE `TestTable` (
  `uniqueId` bigint(20) NOT NULL,
  `CreateTime` datetime NOT NULL,
  `objectDesc` varchar(255) COLLATE utf8mb4_general_ci DEFAULT NULL,
  `objectName` varchar(255) COLLATE utf8mb4_general_ci NOT NULL,
  `UpdateTime` datetime DEFAULT NULL,
  PRIMARY KEY (`uniqueId`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci
```

可以发现已经是`ENGINE=InnoDB`了，上述修改生效。

再次运行测试工程，发现事务回滚生效了。

## 为什么事务对MyISAM引擎不生效呢

为什么Spring的事务无法控制MySQL的MyISAM引擎类型数据表操作呢？很显然，因为MyISAM引擎本身就是非事务安全的，它和InnoDB的介绍概述如下：

> **MyISAM**：这个是默认类型，它是基于传统的ISAM类型，ISAM是Indexed Sequential Access Method （有索引的顺序访问方法) 的缩写，它是存储记录和文件的标准方法。与其他存储引擎比较，MyISAM具有检查和修复表格的大多数工具。 MyISAM表格可以被压缩，而且它们支持全文搜索。它们**不是事务安全的**，而且也不支持外键。**如果事物回滚将造成不完全回滚，不具有原子性**。如果执行大量的SELECT，MyISAM是更好的选择。
> 
> **InnoDB**：这种类型是事务安全的。它与BDB类型具有相同的特性，它们还支持外键。InnoDB表格速度很快。具有比BDB还丰富的特性，因此如果需要一个事务安全的存储引擎，建议使用它。如果你的数据执行大量的INSERT或UPDATE，出于性能方面的考虑，应该使用InnoDB表，**对于支持事物的InnoDB类型的表，影响速度的主要原因是AUTOCOMMIT默认设置是打开的，而且程序没有显式调用BEGIN开始事务，导致每插入一条都自动Commit，严重影响了速度**。可以在执行SQL前调用BEGIN，多条SQL形成一个事物（即使AUTOCOMMIT打开也可以），将大大提高性能。

### 事务的影响范围

先看下测试的代码：

```java
@Transactional(propagation = Propagation.REQUIRED, rollbackFor = Exception.class)
public void executeSave(TestTable testTable) {
    repository.saveAndFlush(testTable);
    System.out.println("save finished");
}
```

调用的代码如下：

```java
@RequestMapping("/saveandflush3")
public String testSaveAndFlushData3(
        @RequestParam(value = "id")String id,
        @RequestParam(value = "name") String name,
        @RequestParam(value = "desc") String desc) {
    TestTable testTable = new TestTable();
    testTable.setId(Long.parseLong(id));
    testTable.setName(name);
    testTable.setDesc(desc);
    transactionTest.executeSave(testTable);
    return "";
}
```

打断点测试发现：当`saveAndFlush`方法执行完之后，数据库中并查询不到记录。只有在`executeSave`方法执行完成跳出此方法之后，即执行到调用逻辑中`return ""`语句的时候，数据库中才能查询到记录。由此可以看出，当`@Transactional`修饰的方法或者类执行结束跳出的时候，也即事务结束。


---

欢迎关注我的公众号“**架构笔录**”，原创技术文章第一时间推送，也可互动一起探讨交流技术。

<center>

   ![](https://raw.githubusercontent.com/veezean/pic_assets/master/assets/comm_pics/contact/gongzhonghao.png)

</center>