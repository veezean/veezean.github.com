---
layout: post
title: "Spring Cloud中Eureka服务管理相关概念"
date: 2019-08-19
categories: SpringCloud
tags: JAVA SpringCloud Eureka
---

* content
{:toc}

`Spring Cloud`算是分布式系统的一系列工具框架集合包。基于提供的这些集合包，可以快速的构建分布式系统。

`Netflix`是Spring Cloud中的重要组件。其中涵盖了一些开箱即用的分布式服务治理能力，诸如`服务管理注册(Eureka)`、`熔断器(Hystrix)`、`智能路由(Zuul)`、`客户端负载均衡(Ribbon)`等等。







## 1 简述

本章主要记录下Netflix中Eureka服务注册管理相关的概念。

## 2 Eureka服务管理

### 2.1 概念介绍

Eureka由两个组件组成：`Eureka服务器`和`Eureka客户端`。Eureka服务器用作服务注册服务器。Eureka客户端是一个java客户端，用来**简化与服务器的交互、作为轮询负载均衡器，并提供服务的故障切换支持**。Netflix在其生产环境中使用的是另外的客户端，它提供基于流量、资源利用率以及出错状态的加权负载均衡。

服务管理的一个大概模型如下所示：

![](/assets/post_pics/SpringCloud-Eureka.md/md_pics_2019-08-14-14-04-27.png)

从结构图上可以看出有一下我们所构建的工程中有`三种`角色：

1. **Eureka Server**: 服务注册中心，负责服务列表的`注册`、`维护`和`查询`等功能;

2. **Service Provider**: 服务提供方，同时也是一个Eureka Client，负责将所提供的服务向Eureka Server进行注册、续约和注销等操作。注册时所提供的主要数据包括`服务名`、`机器ip`、`端口号`、`域名`等，从而能够使服务消费方能够找到;

3. **Service Consumer**: 服务消费方，同时也是一个Eureka Client，同样也会向Eureka Server注册本身所提供的服务。

> 1. `Service Provider(服务提供方)`和`Service Consumer(服务消费方)`并不是一个严格的概念，往往服务消费方也是一个服务提供方，同时服务提供方也可能会调用其它服务方所提供的服务。但是在微服务构建时最好还是遵守业务层级之间的划分，尽量避免服务之间的`循环依赖`。

> 2. 从`Eureka`的视角上看，除了`注册中心`属于`server`节点，`service provider`与`service consumer`均为`client`节点。

### 2.2 Eureka服务端(`注册中心`)

#### 2.2.1 单节点 示例

pom.xml文件中添加对应依赖信息：

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
            <version>1.4.6.RELEASE</version>
        </dependency>
    </dependencies>
```

配置application配置文件信息：

```yml
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    healthcheck:
      enabled: true
    registerWithEureka: false # 作为服务器，禁止将其自身注册到server上去
    fetchRegistry: false      # 是否需要从Eureka Server上拉取注册信息到本地
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/ # 设置与Eureka Server交互的地址，查询服务和注册服务都需要依赖这个地址。默认是http://localhost:8761/eureka ；多个地址可使用 , 分隔。
```

上述几个关键配置项的含义说明如下：
> * `eureka.client.register-with-eureka`：表示是否将自己注册到Eureka Server，默认为true。
> * `eureka.client.fetch-registry`：表示是否从Eureka Server获取注册信息，默认为true。
> * `eureka.client.serviceUrl.defaultZone`：设置与Eureka Server交互的地址，查询服务和注册服务都需要依赖这个地址。默认是http://localhost:8761/eureka，多个地址可使用`,`分隔。

按照spring boot的要求指定application类：

```java {.line-numbers, highlight=[2,5]}
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        new SpringApplicationBuilder(EurekaServerApplication.class).web(true).run(args);
    }
}
```

在浏览器中，输入`http://localhost:8761/`可以查看到eureka的管理界面，可以看到里面的服务列表等信息（示例中没有注册任何服务，因此看到列表是空的），如下：

![](/assets/post_pics/SpringCloud-Eureka.md/md_pics_2019-08-14-14-56-59.png)

#### 2.2.2 保障`高可用性`实现

注册中心这么关键的服务，如果是单点话，遇到故障就是毁灭性的。

在一个分布式系统中，服务注册中心是最重要的基础部分，理应随时处于可以提供服务的状态。为了维持其可用性，使用集群是很好的解决方案。

在上面单个节点示例的基础上，修改application.yml，配置2个server节点的属性信息，并且两个eureka server节点`相互注册`：

```yml {.line-numbers}
---
spring:
  application:
    name: spring-cloud-eureka1 # 注册到注册中心的服务名称
  profiles: peer1
server:
  port: 8761
eureka:
  instance:
    hostname: peer1
  client:
    healthcheck:
      enabled: true
      registerWithEureka: true # 集群高可用场景，作为服务器，将其自身注册到server上去
    fetchRegistry: true        # 是否需要从Eureka Server上拉取注册信息到本地
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:8762/eureka/

---
spring:
  application:
    name: spring-cloud-eureka2  # 注册到注册中心的服务名称
  profiles: peer2
server:
  port: 8762
eureka:
  instance:
    hostname: peer2
  client:
    healthcheck:
      enabled: true
      registerWithEureka: true # 集群高可用场景，作为服务器，将其自身注册到server上去
    fetchRegistry: true        # 是否需要从Eureka Server上拉取注册信息到本地
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:8761/eureka/
```

上面的配置中，配置了2个profile属性段，名称分别为peer1和peer2,且两个节点的`eureka.client.serviceUrl.defaultZone`属性值相互配置为对方的地址。

**特别注意**：

> 对于集群高可用方案中，必须要配置`registerWithEureka`和`fetchRegistry`都为`true`，多个Server节点之间才会**同步各自的注册数据信息**。

##### 如何本地启动两个SpringBoot进程

###### 编译jar然后cmd启动

通过`mvn clean package`命令编译并生成jar包，然后通过如下命令，分别启动2个server进程：
```
java -jar com.vzn.localtest.springcloud.eureka-server-1.0-SNAPSHOT.jar --spring.profiles.active=peer1
```
```
java -jar com.vzn.localtest.springcloud.eureka-server-1.0-SNAPSHOT.jar --spring.profiles.active=peer2
```
上述命令中，通过`spring.profiles.active`属性，指定进程运行的时候，加载哪个application属性（前面已经写明，通过application.yml方式将多个server的配置写在了一个文件里，启动进程的时候需要指明分别使用那个配置块）

###### 通过Idea直接启动

在Idea工具中，右上角应用名称选择的地方，下拉，选择`edit configurations`并点击：

![](/assets/post_pics/SpringCloud-Eureka.md/md_pics_2019-08-19-13-31-49.png)

在打开的窗口中，左上角点击`+`新增一个启动项设置，指定`Main Class`对应的启动类，并在`Program Arguments`一栏中填入`--spring.profiles.active=peer1`，同样的方式可以继续新增启动项，`Program Arguments`一栏中填入`--spring.profiles.active=peer2`，如下所示：

![](/assets/post_pics/SpringCloud-Eureka.md/md_pics_2019-08-19-13-30-19.png)

配置完成，启动的时候，选择对应的启动项，直接点击右侧的`绿色启动按钮`即可，如下：

![](/assets/post_pics/SpringCloud-Eureka.md/md_pics_2019-08-19-13-36-09.png)

##### 查看效果

此处示例中配置的是`hostname`的方式，调测的时候，在host文件中，加上：

```
127.0.0.1 peer1
127.0.0.1 peer2
```

在浏览器中输入`http://127.0.0.1:8761/`和`http://127.0.0.1:8762/`，可以发现peer1和peer2两个进程已经相互注册了：

![](/assets/post_pics/SpringCloud-Eureka.md/md_pics_2019-08-14-16-02-42.png)

![](/assets/post_pics/SpringCloud-Eureka.md/md_pics_2019-08-14-16-03-16.png)

通过上述简单的示例可以看出，Eureka通过互相注册的方式来实现高可用的部署，只需要将Eureke Server配置其他可用的serviceUrl就能实现高可用部署。

在生产中可能需要`三台`或者`大于三台`的注册中心来保证服务的稳定性，配置的原理其实都一样，将注册中心分别指向其它的注册中心。Eureka会在有关联的各个节点之间进行`数据同步`，保证集群高可用。

### 2.3 Eureka客户端(`服务提供方`与`服务消费方`)

#### 2.3.1 实现方式

与Eureka Server实现原理相似，只是使用的是`@EnableEurekaClient`注解，如下示例代码的第2行所示。

```java {.line-numbers, highlight=[2]}
@SpringBootApplication
@EnableEurekaClient
public class EurekaServerApplication {
    public static void main(String[] args) {
        new SpringApplicationBuilder(EurekaServerApplication.class).web(true).run(args);
    }
}
```

在一个**常规的SpringBoot应用**的Application启动类上添加`@EnableEurekaClient`标识，并在`application.properties`（或者其它形式的比如`xml`\\`yml`等格式）配置文件中添加上Eureka相关的配置，即可实现将自身注册到Eureka Server注册中心去。

将应用注册到注册中心的时候，可以在application配置文件中配置对应的`service name`值，单个`service name`值可以*被多个进程同时注册*，表示**此服务具有多个集群节点**。

如下所示：

```{.line-numbers}
---
spring:
  application:
    name: eureka-client-provier1
  profiles: peer1
server:
  port: 8763
eureka:
  instance:
    hostname: peer1
  client:
    healthcheck:
      enabled: true
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:8762/eureka/

---
spring:
  application:
    name: eureka-client-provier2
  profiles: peer2
server:
  port: 8764
eureka:
  instance:
    hostname: peer2
  client:
    healthcheck:
      enabled: true
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:8761/eureka/

---
spring:
  application:
    name: eureka-client-consumer3
  profiles: peer3
server:
  port: 8765
eureka:
  instance:
    hostname: peer2
  client:
    healthcheck:
      enabled: true
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:8761/eureka/
```

启动Client进程，然后到Server对应的管理界面查看，可以看到已经注册到管理中心了，注册名称对应配置文件中的`spring.application.name`值：

![](/assets/post_pics/SpringCloud-Eureka.md/md_pics_2019-08-19-15-00-30.png)

#### 2.3.2 实现负载均衡调用服务

结合`Ribbon负载均衡器`，通过`service name`的方式访问对应的`service provider`应用，可以实现客户端的请求在`service name`对应的多个注册的进程之间`负载均衡`。

实现负载均衡的方式较为简单，在`RestTemplate`的注入初始化的地方，添加上`@LoadBalanced`注解即可实现：

```java {.line-numbers, highlight=[5]}
@Configuration
public class EurekaConfiguration {

    @Bean
    @LoadBalanced
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

注意在访问service的时候，需要使用`service name`的方式去访问，才能达到`负载均衡`的效果。

### 2.4 Eureka的自我保护与心跳机

测试demo中，发现client进程停止后，server注册列表中依旧存在此client的信息，了解了下，是和Eureka的自我保护以及心跳等有关系。

#### Eureka自我保护机制

有的时候，Eureka Server的页面中，会出现如下图所示的红色字体描述：

![](/assets/post_pics/SpringCloud-Eureka.md/md_pics_2019-08-19-15-37-48.png)

这是因为Eureka开启了自我保护机制。

按照通常的认知，如果Eureka Server在一定时间内没有接收到某个微服务实例的心跳时，EurekaServer应该会注销该实例（默认90s）。但是，假设微服务进程正常运行，只是网络故障等原因，导致服务与注册中心之间的网络不通，心跳没有正常送达（其实这个时候服务是正常可访问的），如果注册中心将其移除掉，会导致consumer无法再发现此服务，可能会导致业务上面的问题。

鉴于上述场景可能的问题，Eureka Server提供了自我保护机制，会在此种场景下禁止将注册的service移除，来防止了此类问题的出现。当网络恢复的时候，会自动退出自我保护机制。

当然，也可以在配置文件中关闭Server的自我保护机制，配置项如下：

```properties {.line-numbers}
#关闭Eureka服务自我保护
eureka.server.enable-self-preservation=false
#服务端每隔n秒刷新依次服务列表，将无效服务剔除
eureka.server.eviction-interval-timer-in-ms=5000
```

关闭后，界面上会有对应的提示说明：

![](/assets/post_pics/SpringCloud-Eureka.md/md_pics_2019-08-19-15-57-48.png)

再次尝试client先启动注册到Server之后，再停止Client进程，等待一段时间之后刷新Server界面，会发现Server的注册列表中，自动移除了已经停止进程的client的注册信息：

![](/assets/post_pics/SpringCloud-Eureka.md/md_pics_2019-08-19-16-09-36.png)

#### 客户端心跳

Eureka使用客户端心跳维持更新注册状态信息。默认情况下`eureka.instance.leaseRenewalIntervalInSeconds`值为30，如果有需要的话，可以修改此值来加快服务状态变更的速度（最好保持30s别动）。

添加client端的配置信息如下：

```properties
leaseRenewalIntervalInSeconds: 10    # 客户端往服务器端发送心跳的间隔时间,默认是30s，修改小一些可以加快被发现速度
leaseExpirationDurationInSeconds: 60 # 客户端多久不发消息给注册中心，注册中心直接将客户端的注册信息移除。默认90s
```

当一个服务的注册状态发生变更的时候，需要`3个心跳周期`，才能保证客户端、服务器端、本地三者的元数据全部更新一致。

#### 二者关联

<font color="red">当修改了client对应的心跳默认配置的时候，会打破server的自我保护机制</font>，即当心跳超过自定义的时间之后，服务器会直接移除此client的注册信息。

### 2.5 Eureka Server与Client交互的全流程

> 1. EurekaServer 提供服务发现的能力，各个微服务启动时，会向EurekaServer注册自己的信息（例如：ip、端口、微服务名称等），EurekaServer会存储这些信息；
> 2. EurekaClient是一个Java客户端，用于简化与EurekaServer的交互；
> 3. 微服务启东后，会定期性（默认30s）的向EurekaServer发送心跳以续约自己的“租期”；
> 4. 如果EurekaServer在一定时间内未接收某个微服务实例的心跳，EurekaServer将会注销该实例（默认90s）；
> 5. 默认情况下，EurekaServer同时也是EurekaClient。多个EurekaServer实例，互相之间通过复制的方式，来实现服务注册表中数据的同步；
> 6. EurekaClient也会缓存服务注册表中的信息。


## 3 过程问题记录

启动报错：

```
12:23:15.304 [main] ERROR org.springframework.boot.SpringApplication - Application run failed
java.lang.NoSuchMethodError: org.springframework.boot.builder.SpringApplicationBuilder.<init>([Ljava/lang/Object;)V
	at org.springframework.cloud.bootstrap.BootstrapApplicationListener.bootstrapServiceContext(BootstrapApplicationListener.java:161)
	at org.springframework.cloud.bootstrap.BootstrapApplicationListener.onApplicationEvent(BootstrapApplicationListener.java:102)
	at org.springframework.cloud.bootstrap.BootstrapApplicationListener.onApplicationEvent(BootstrapApplicationListener.java:68)
	at org.springframework.context.event.SimpleApplicationEventMulticaster.doInvokeListener(SimpleApplicationEventMulticaster.java:172)
	at org.springframework.context.event.SimpleApplicationEventMulticaster.invokeListener(SimpleApplicationEventMulticaster.java:165)
	at org.springframework.context.event.SimpleApplicationEventMulticaster.multicastEvent(SimpleApplicationEventMulticaster.java:139)
	at org.springframework.context.event.SimpleApplicationEventMulticaster.multicastEvent(SimpleApplicationEventMulticaster.java:127)
	at org.springframework.boot.context.event.EventPublishingRunListener.environmentPrepared(EventPublishingRunListener.java:75)
```

原因：

由于Spring Boot和Spring Cloud之间的版本不匹配导致的，修改pom.xml中两个的版本相匹配，即可。


## 4 资料汇总

* [SpringCloud中文介绍](https://springcloud.cc/spring-cloud-dalston.html)

  对Spring Cloud集合内相关组件做了个相对浅显的介绍，入门适用。

* [如何通过idea将SpringBoot应用导出为jar](https://blog.csdn.net/m0_37202351/article/details/81738357)

  配置可以大概参考下，但是实测的时候，这种方式打出来的包启动的时候会报错，所以最简单的一种方式，直接通过mvn clean package命令打包即可。

* [Spring Cloud Eureka使用简单示例教程](https://www.jianshu.com/p/0325c5e8d427)

* [Spring Cloud Eureka 心跳相关介绍](https://www.jianshu.com/p/36856d2a7847)

* [关于Eureka的注册与发现](https://www.cnblogs.com/lfalex0831/p/9184428.html)

---

欢迎关注我的公众号“**架构笔录**”，原创技术文章第一时间推送，也可互动一起探讨交流技术。

<center>

   ![](https://raw.githubusercontent.com/veezean/pic_assets/master/assets/comm_pics/contact/gongzhonghao.png)

</center>