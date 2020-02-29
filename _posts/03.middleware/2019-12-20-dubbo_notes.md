---
layout: post
title: "Dubbo！用更优雅的方式来实现RPC调用吧"
date: 2019-12-20
categories: SpringBoot
tags: SpringBoot Spring LogBack 日志 SLF4J JAVA
---

* content
{:toc}

Dubbo是阿里开源的一个RPC调用框架。可以大大简化RPC远程调用实现复杂度，使得可以更专注于业务能力的实现，可以向本地API调用一样使用远程服务接口。

本文主要是对Dubbo的一些关键信息进行记录，并演示下代码中使用Dubbo的具体方法。





## 1 Dubbo关键概念的说明

### 1.1 DUBBO基本原理

Dubbo的一个基本的组网以及原理图如下（图片摘自网络）：


![](\assets/post_pics/2019-12-20-dubbo_notes.md/md_pics_2019-12-26-17-39-14.png)

**节点角色说明：**

> Provider: 暴露服务的服务提供方。 
> Consumer: 调用远程服务的服务消费方。 
> Registry: 服务注册与发现的注册中心。 
> Monitor: 统计服务的调用次调和调用时间的监控中心。 

更多新详细的说明，可以参见dubbo[官方文档](http://dubbo.apache.org/zh-cn/index.html)，对于细节描述，本文不再赘述。


### 1.2 DUBBO对协议的支持

**1. Dubbo可以选择协议**

> 1) `Dubbo`协议，采用`NIO`复用单一长连接，并使用`线程池`并发处理请求，减少握手和加大并发效率，性能较好**（推荐使用）**【问题：在大文件传输时，单一连接会成为瓶颈】
> 2) `Rmi`协议，可与原生`RMI`互操作，基于`TCP`协议。【问题：偶尔会连接失败，需重建Stub】
> 3) `Hessian`协议，可与原生的`Hessian`互操作，基于`HTTP`协议【需要hessian.jar支持，http短连接的开销大】

**2. Dubbo可选用多种序列化方式**

> 1) `Hessian Serialization`，性能较好，多语言支持【问题：Hessian各版本兼容不好】
> 2) `Dubbo Serialization`，通过不传输POJO的类元信息，在大量POJO传输时，性能好【问题：当参数对象增加字段时，需外部文件声明】
> 3) `Json Serialization`，纯文本，可跨语言解析，缺省采用FastJson解析【问题：性能较差】
> 4) `Java Serialization`，java原生支持【问题：性能较差】

**3. 默认协议为Dubbo，序列化为Hessian**


### 1.3 服务Port和dubbo协议Port之间的区别

进程启动后会有一个对应的port，比如SpringBoot的服务，在application.properties中会设定一个`server.port`值，这个port通常是指的`http协议的对外访问端口号`。

`dubbo`作为一个独立协议，对外提供服务的时候，也需要指定一个`dubbo协议对应的port`信息。两个端口要**区分开**，不是同一个，这一点，初学者很容易弄混淆。

`dubbo`的`port`指定方式有两种：

* 1. XML中指定
```xml
<!-- 用dubbo协议在20880端口暴露服务 -->
 <dubbo:protocol name="dubbo" port="20880" ></dubbo:protocol>
```

* 2. SpringBoot的工程，代码中指定
```java
    @Bean
    public ProtocolConfig protocolConfig() {
        ProtocolConfig protocolConfig = new ProtocolConfig();
        protocolConfig.setPort(8811);
        protocolConfig.setName("dubbo");
        return protocolConfig;
    }
```

## 2 Dubbo服务的代码实现

复杂、大型分布式系统中，一般都需要引入服务注册中心，Dubbo默认是配套Zookeeper作为注册中心，也可以对接其它的分布式服务注册中心。这里比较推荐使用阿里开源的Nacos作为Dubbo配套的注册中心，详细请参见我的另一篇文章：[《玩转Nacos！替代Eureka作为配置中心与注册中心》](https://veezean.github.io/2020/02/24/nacos_notes/)

下面的示例代码中，没有使用注册中心，演示消费者直连服务提供者进行调用的场景，主要侧重点在于说明如何集成与使用Dubbo。

### 2.1 SpringBoot集成Dubbo对应的Maven依赖

需要引入dubbo包（示例，没有使用注册中心，所以不需要引入zookeeper相关包）。

```xml
    <dependency>
        <groupId>com.alibaba.spring.boot</groupId>
        <artifactId>dubbo-spring-boot-starter</artifactId>
        <version>2.0.0</version>
    </dependency>
```
### 2.2 Service Provider服务提供方

#### 2.2.1 实现定义的需要对外开放的RPC接口，实现具体的实现逻辑

关注两个注解：
1、`@Component`  这个就是按照SpringBoot的逻辑，将服务自动Spring托管，完成初始化加载；
2、`@com.alibaba.dubbo.config.annotation.Service`  这个是Dubbo提供，表示此类是需要通过dubbo对外提供的服务。

如下所示：

```java
@Component
@com.alibaba.dubbo.config.annotation.Service(timeout = 5000)
public class RpcDemoServiceImpl implements RpcDemoService {

    @Override
    public String queryCurrentTime(String queryId) {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS");
        String formattedDate = simpleDateFormat.format(new Date());
        System.out.println(queryId + " called, current time is: " + formattedDate);
        return queryId + "|" + formattedDate;
    }
}
```

#### 2.2.2 启动类中定义需要扫描的dubbo目录

关注2个注解的含义：
1、`@SpringBootApplication`  指定按照一个常规的SpringBoot应用进行启动；
2、`@EnableDubbo`  开启对Dubbo的支持，并指定对应目录，将扫描指定的目录下的类，找到所有被`@com.alibaba.dubbo.config.annotation.Service`标识的需要对外发布的dubbo服务接口类。

如下所示：
```java
@SpringBootApplication
@EnableDubbo(scanBasePackages = {"com.vzn.local.dubbo.provider.service"})
public class ProviderApp {
    public static void main(String[] args) {
        SpringApplication.run(ProviderApp.class);
    }
}
```

#### 2.2.3 指定dubbo相关设置

两种方式，可以在代码中直接写、也可以在配置文件中指定。

##### 2.2.3.1 通过代码中Bean注入的方式指定

通过`@Bean`的方式，注入Dubbo的关键定义。与XML配置的内容相同，此处通过对应的JAVA对象的方式，设定具体的值。

主要的几个配置类有：
1、 ApplicationConfig
    对应XML中的`<dubbo:application />`节点的配置信息。

2、 ProtocolConfig
    对应XML中的`<dubbo:protocol />`节点的配置信息。

3、 RegistryConfig
    对应XML中的`<dubbo:registry />`节点的配置信息。


示例代码如下：

```java
@Configuration
public class DubboConfig {

    @Bean
    public ApplicationConfig applicationConfig() {
        ApplicationConfig applicationConfig = new ApplicationConfig();
        applicationConfig.setName("dubbo-provider");
        return applicationConfig;
    }

    @Bean
    public ProtocolConfig protocolConfig() {
        ProtocolConfig protocolConfig = new ProtocolConfig();
        protocolConfig.setPort(8811);
        protocolConfig.setName("dubbo");
        return protocolConfig;
    }

    @Bean
    public RegistryConfig registryConfig() {
        RegistryConfig registryConfig = new RegistryConfig();
        registryConfig.setAddress("N/A");
        registryConfig.setRegister(false);

//        registryConfig.setAddress("zookeeper://127.0.0.1:2181");
        return registryConfig;
    }
}
```

##### 2.2.3.2 通过`application.properties`中配置

```properties
spring.application.name=dubbo-provider
server.port=28811

# dubbo config
dubbo.application.id=dubbo-provider
dubbo.application.name=dubbo-provider
dubbo.protocol.name=dubbo
dubbo.protocol.port=8811
dubbo.registry.address=N/A
```


至此，一个简单的SpringBoot中通过代码与注解的方式集成dubbo的逻辑实现完毕，直接启动即可。

注意： 
> 1. 本地测试DEMO，没有设置注册中心，采用直连的方式来进行调用，因此RegistryConfig中配置的address为`N/A`；
> 2. 实际使用时，如果使用`zookeeper`作为注册中心，则此处配置为`zookeeper`对应的`IP`和`port`值。(当然，**也可以选择`Nacos`作为配置+注册中心进行使用，推荐**)

### 2.3 Service Consumer服务消费方

#### 2.3.1 本地引用RPC服务接口，实现对应service类

实现一个简单的SpringBoot的工程，在常规的`@Service`标识的类中，引入远程RPC服务对应的本地API接口，并通过`@com.alibaba.dubbo.config.annotation.Reference`注解进行标识，此注解即表示此bean将通过RPC的方式调用到远端真实的实现类中的方法，并返回执行结果。

这个就是Dubbo框架的一个优势，**将所有的底层逻辑都给封装屏蔽了，使得业务代码中使用RPC的接口时，可以按照本地API的方式进行直接调用，极大的降低了编码难度，使得开发者可以更专注于自身业务，而无需关注具体应该如何RPC调用到远程服务端**。

示例代码如下：

```java
@org.springframework.stereotype.Service
public class RpcConsumerService {

//    @Reference
//    private RpcDemoService rpcDemoService;

    // 下面两种方式指定的参数，都可以调用成功
    @Reference(protocol = "dubbo", url = "172.31.236.79:8811", interfaceClass = RpcDemoService.class)
    // @Reference(url = "dubbo://172.31.236.79:8811")
    private RpcDemoService rpcDemoService;

    public String getTime(String requestId) {
        return rpcDemoService.queryCurrentTime(requestId);
    }
}
```

说明：
> 本地DEMO代码，没有注册中心，采用直连方式，所以上面`@Reference`中指定了对应的url信息（直连服务具体的IP和dubbo协议port）。

#### 2.3.2 启动类中开启dubbo的支持

如下：

```java
@SpringBootApplication
@EnableDubbo(scanBasePackages = {"com.vzn.local.dubbo.consumer.service"})
public class ConsumerApp {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApp.class);
    }
}
```

#### 2.3.3 代码中手动注入Dubbo相关的配置信息

如下，和provider类似，但略有不同：

```java
@Configuration
public class DubboConfig {

    @Bean
    public ApplicationConfig applicationConfig() {
        ApplicationConfig applicationConfig = new ApplicationConfig();
        applicationConfig.setName("dubbo-consumer");
        return applicationConfig;
    }

    @Bean
    public RegistryConfig registryConfig() {
        RegistryConfig registryConfig = new RegistryConfig();
        registryConfig.setAddress("N/A");
//        registryConfig.setAddress("zookeeper://127.0.0.1:2181");
        registryConfig.setClient("curator");
        return registryConfig;
    }

    @Bean
    public ConsumerConfig consumerConfig() {
        ConsumerConfig consumerConfig = new ConsumerConfig();
        consumerConfig.setTimeout(3000);
        return consumerConfig;
    }
}
```

或者直接在`application.properties`中配置：
```properties
spring.application.name=dubbo-consumer
server.port=28812

# dubbo config
dubbo.application.id=dubbo-consumer
dubbo.application.name=dubbo-consumer
dubbo.protocol.name=dubbo
dubbo.registry.address=N/A
dubbo.registry.client=curator

dubbo.server=false
dubbo.consumer.timeout=3000
```

消费者服务中，增加了一个`ConsumerConfig`的配置，本示例中简单定义了个调用端超时时间值。

#### 2.3.4 提供个controller服务，用于测试调用

按照SpringBoot的常规实践，提供一个controller服务，用于触发测试使用。

```java
@RestController
@RequestMapping("/demo/dubbo")
public class RpcController {

    @Resource(name = "rpcConsumerService")
    private RpcConsumerService service;

    @GetMapping("/get/time/{requestId}")
    public String getTime(@PathVariable("requestId") String requestId) {
        String time = service.getTime(requestId);
        System.out.println("RPC call the getTime result:" + time);
        return time;
    }
}
```

至此，一个简单的SpringBoot实现的Dubbo客户端服务就实现完成了，启动后，调用controller服务，即可验证效果。

### 2.4 测试调用验证

先后启动Provider和Consumer服务，然后在浏览器中发送通知触发consumer服务的方法：

```
http://127.0.0.1:28812/demo/dubbo/get/time/1
```

测试结果：
```
1|2019-12-23 14:21:11.463
```

可以发现，在consumer中的方法执行时，从consumer进程中RPC调用到了provider进程中对应的方法，并返回了方法具体的执行结果，测试验证完毕。
