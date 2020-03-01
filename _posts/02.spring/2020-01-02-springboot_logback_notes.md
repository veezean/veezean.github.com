---
layout: post
title: "Just Do IT，你的SpringBoot日志输出格式，由你来定！"
date: 2020-01-02
categories: SpringBoot
tags: SpringBoot Spring LogBack 日志 SLF4J JAVA
---

* content
{:toc}

`LogBack`是默认集成在`SpringBoot`里面的，是基于`Slf4j`的日志框架。**默认情况下，SpringBoot是按照`INFO`级别输出到控制台的**。

日志级别顺序如下：
```
ALL < TRACE < DEBUG < INFO < WARN < ERROR < OFF
```
在实际项目中，通常需要将日志信息落盘保存，因此需要添加对应配置文件。





## SpringBoot中Logback配置

### 1 添加自定义的配置文件

`LogBack`可以直接在`application.properties`或`application.yml`中配置，但仅支持一些简单的配置，复杂的文件输出还是需要配置在 xml 配置文件中。

有两种选择：
* **1、使用固定的logback-spring.xml作为名称，无需额外配置即可**

> logback启动时会尝试在classpath目录中查找`logback-test.xml`文件；如果文件不存在，则查找`logback.xml`文件；如果还不存在，则会`自动配置`，使用默认配置。
> 
> 与`SpringBoot`结合时，官方推荐使用的xml名字的格式为：`logback-spring.xml`而不是logback.xml，因为带spring后缀的可以使用<springProfile>这个标签。
> 
> 所以，在resource下创建`logback-spring.xml`文件，然后在此xml中进行配置，即可。

* **2、使用自定义的配置文件名称，需要额外的指定日志配置的文件路径信息**

> 如果没有使用上面1中体积的默认名称，则*需要在配置文件中指定需要加载的日志配置文件路径名称*信息。
> 
> 示例如下：
> 
> ![](\assets/post_pics/2020-01-02-springboot_logback_notes.md/md_pics_2020-01-02-14-10-22.png)

### 2 Logback配置文件示例与含义解说

LogBack配置中可以对日志输出情况做一些个性化的定制，主要支持如下能力：

> 1. 支持读取application.yml或者application.properties中的属性；
> 2. 支持根据不同的spring profile类型，实现不同的log输出配置；
> 3. 支持日志输出到文件中根据级别进行的过滤；
> 4. 支持日志文件的绕接、定时备份、超时清理、超限清理等机制；
> 5. 支持日志输出内容的定制。

各种关键配置的配置方式以及含义说明，参见如下的一个配置示例，注释上有详细的说明。

对于pattern的含义说明，详细信息可以参见[http://logback.qos.ch/manual/layouts.html](http://logback.qos.ch/manual/layouts.html)说明。


```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 日志级别从低到高分为TRACE < DEBUG < INFO < WARN < ERROR < FATAL，如果设置为WARN，则低于WARN的信息都不会输出 -->
<!-- scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true -->
<!-- scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。 -->
<!-- debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。 -->
<configuration scan="true" scanPeriod="10 seconds">
  <!-- ERROR日志格式,后面在对应的appender定义中指定pattern为此值，即可以按照此处的日志格式进行输出 -->
  <property name="FILE_ERROR_PATTERN"
            value="${FILE_LOG_PATTERN:-%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} %file:%line: %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>

  <!-- 彩色日志 -->
  <!-- 彩色日志依赖的渲染类 -->
  <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
  <conversionRule conversionWord="wex"
                  converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>
  <conversionRule conversionWord="wEx"
                  converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"/>

  <!-- 彩色日志格式,后面在对应的appender定义中指定pattern为此值，即可以按照此处的日志格式进行输出  -->
  <!-- 默认情况下， 自带的CONSOLE_LOG_PATTERN属性，即为彩色的，此处重新定义了个新的属性，用于演示彩色控制台的使用示例-->
  <property name="CONSOLE_LOG_PATTERN_COLOR"
            value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{0}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>

  <!-- 解说下具体的pattern中属性的含义,详细参见http://logback.qos.ch/manual/layouts.html-->
  <!-- %logger{x}，或者写为%c{x}，表示类名信息，如果x为0则表示仅输出类名，如果x大于0则尝试输出全路径，并按照指定的x值进行缩写。
      比如com.test.sample.Main：
      设置%logger{0}为Main
      设置%logger{5}可缩写为c.t.s.Main
      设置%logger{20}为com.test.sample.Main-->
  <!--  %d表示日期信息(或者%date)，默认格式为【yyyy-MM-dd HH:mm:ss,SSS】，可以通过%d{x}自定义格式，比如%d{yyyy-MM-dd HH:mm:ss.SSS}-->
  <!--  %p表示日志的level,或者%le\%level-->
  <!--  %L表示相对应的行号，或者%line-->
  <!--  %m表示实际的需要输出的日志内容字符串,或者%msg\%message都可以-->
  <!--  %replace(p){r,t}表示对p中给定的内容进行字符串替换，将r替换为t，支持正则，比如将p中的换行替换为下划线，防止日志攻击-->
  <!--  %M表示方法名称，或者%method也可以-->
  <!--  %t表示线程名称，或者%thread也可以-->
  <!-- %ex{x}用于指定错误异常堆栈的打印策略，x可以为short\full或者具体数字，表示打印多少行堆栈-->
  <!--  %n表示换行-->
  <!-- 字符串对齐、截取、格式化说明：
       在上述的各个占位符中，在%和具体字符之间，可以插入格式化指令，以%c为例，如下：
       %20c 表示%c的内容如果不足20位，则在左侧以空格填充满20长度
       %-20c 与%20c相似，区别在于，会在右侧以空格填满20长度
       %.20c 表示%c内容如果超过20，则会截取掉开头的内容，只留下右侧20位长度
       %.-20c 与%.20c相似，区别在于，会截取掉末尾的内容，只留下开头20位长度
       以此类推，还可以组合出如下使用方式：
       %20.30c 表示最短20位，最长30位，如果不足20位则左侧补齐空格，如果超过30位则丢弃左侧开头的字符串
       %-20.30c 和%20.30c类似，区别在于不足20位的时候，在右侧补齐空格
       %-20.-30c 和%-20.30c类似，区别在于超过30位的时候会丢弃结尾部分的字符串
       %20.-30c 和20.30c类似，区别在于超过30位的时候会丢弃结尾部分的字符串
      -->

  <property name="SELF_DEFINE_LOG_PATTERN"
            value="[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%5.5p] [%5.10t] [%30.30c{0}.%20.20M:%4.4L] -> %replace(%.-300m){'\r\n','__'}%ex{full}%n"/>

  <!-- Spring Boot 提供了一个默认的 xml 配置，可以按照如下方式引入 -->
  <include resource="org/springframework/boot/logging/logback/defaults.xml"/>

  <!-- 引入application.yml配置文件，后续可以直接引用里面的属性值-->
  <!-- 注意：在此文件中直接${}的方式引用application.yml里面配置可能会报错.
  因为logback.xml的加载顺序早于springboot的application.yml (或application.properties) 配置文件。
  所以可以先springProperty的方式定义个本地变量引入进来，再进行引用此本地变量-->
  <!-- 可以将一些公共的内容放到application.yml里面去配置，然后此文件中引用，后续可以避免修改此xml，简单的参数直接修改下application.yml就行了-->
  <!-- 比如将日志存放路径与文件大小信息从配置中读取，这样dev和prod可以指定不同的逻辑-->
  <property resource="application.yml"/>
  <springProperty scope="context" name="LOG_HOME" source="selfdefine.logfile.rootPath"/>
  <springProperty scope="context" name="LOG_FILE_SIZE" source="selfdefine.logfile.max-size"/>

  <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
      <level>INFO</level>
    </filter>

    <!-- 指定此append对应的日志内容的格式-->
    <encoder>
      <pattern>${CONSOLE_LOG_PATTERN_COLOR}</pattern>
      <charset>UTF-8</charset>
    </encoder>

  </appender>

  <appender name="CONSOLE2" class="ch.qos.logback.core.ConsoleAppender">
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
      <level>INFO</level>
    </filter>

    <!-- 指定此append对应的日志内容的格式-->
    <encoder>
      <pattern>${SELF_DEFINE_LOG_PATTERN}</pattern>
      <charset>UTF-8</charset>
    </encoder>

  </appender>

  <appender name="FILE_INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <!--如果只是想要 Info 级别的日志，只是过滤 info 还是会输出 Error 日志，因为 Error 的级别高， 所以我们使用下面的策略，可以避免输出 Error 的日志-->
    <!-- 指定过滤的日志级别，并可以额外指定是拒绝还是允许。如果仅配置了level属性，则表示只有高于等于level的值才会记录，其余丢弃（参考下面FILE_ERROR对应appender）-->
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
      <!--过滤 Error-->
      <level>ERROR</level>
      <!--匹配到就禁止-->
      <onMatch>DENY</onMatch>
      <!--没有匹配到就允许-->
      <onMismatch>ACCEPT</onMismatch>
    </filter>
    <!--日志名称，如果没有File 属性，那么只会使用FileNamePattern的文件路径规则如果同时有<File>和<FileNamePattern>，那么
    当天日志是<File>，明天会自动把今天的日志改名为今天的日期。即，<File> 的日志都是当天的。-->
    <!--<File>logs/info.spring-boot-demo-logback.log</File>-->
    <!--滚动策略，按照时间滚动 TimeBasedRollingPolicy-->
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!--文件路径，支持相对路径或者绝对路径（尽量避免相对路径，通过绝对路径保证存储位置的固定）,定义了日志的切分方式——把每一天的日志归档到一个文件中,以防止日志填满整个磁盘空间-->
      <!-- 指定文件的路径以及对应的文件命名格式，其中%i表示递增标识ID序号，日志切换绕接的时候递增-->
      <FileNamePattern>${LOG_HOME}/spring-boot-demo-logback/info_%d{yyyy-MM-dd}.part_%i.log</FileNamePattern>
      <!--只保留最近90天的日志-->
      <maxHistory>90</maxHistory>
      <!--用来指定日志文件的上限大小，那么到了这个值，就会删除旧的日志-->
      <!--<totalSizeCap>1GB</totalSizeCap>-->
      <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
        <!-- maxFileSize:这是活动文件的大小，默认值是10MB,本篇设置为1KB，只是为了演示 -->
        <!--此处演示从application.yml中读取配置的属性值，通过${}的方式引用-->
        <maxFileSize>${LOG_FILE_SIZE}</maxFileSize>
      </timeBasedFileNamingAndTriggeringPolicy>
    </rollingPolicy>
    <!--<triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">-->
    <!--<maxFileSize>1KB</maxFileSize>-->
    <!--</triggeringPolicy>-->

    <!-- 指定此append对应的日志内容的格式-->
    <encoder>
      <pattern>${FILE_LOG_PATTERN}</pattern>
      <charset>UTF-8</charset> <!-- 此处设置字符集 -->
    </encoder>
  </appender>

  <appender name="FILE_ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <!--如果只是想要 Error 级别的日志，那么需要过滤一下，默认是 info 级别的，ThresholdFilter-->
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
      <!-- 指定过滤的日志级别，只有等于或者高于此级别的，才会通过此appender进行输出-->
      <level>Error</level>
    </filter>
    <!--日志名称，如果没有File 属性，那么只会使用FileNamePattern的文件路径规则如果同时有<File>和<FileNamePattern>，那么当天日志是<File>，明天会自动把今天的日志改名为今天的日期。即，<File> 的日志都是当天的。-->
    <!--<File>logs/error.spring-boot-demo-logback.log</File>-->
    <!--滚动策略，按照时间滚动 TimeBasedRollingPolicy-->
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!--文件路径,定义了日志的切分方式——把每一天的日志归档到一个文件中,以防止日志填满整个磁盘空间-->
      <FileNamePattern>logs/spring-boot-demo-logback/error.created_on_%d{yyyy-MM-dd}.part_%i.log</FileNamePattern>
      <!--只保留最近90天的日志-->
      <maxHistory>90</maxHistory>
      <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
        <!-- maxFileSize:这是活动文件的大小，默认值是10MB,本篇设置为1KB，只是为了演示 -->
        <maxFileSize>2MB</maxFileSize>
      </timeBasedFileNamingAndTriggeringPolicy>
    </rollingPolicy>

    <!-- 指定此append对应的日志内容的格式-->
    <encoder>
      <pattern>${FILE_ERROR_PATTERN}</pattern>
      <charset>UTF-8</charset> <!-- 此处设置字符集 -->
    </encoder>
  </appender>

  <!-- 指定日志输出到哪些位置、以及root日志的输出level-->
  <!-- 默认情况下的使用，任何spring profile值情况下都会使用下面的配置，即输出到console中-->
  <root level="info">
    <appender-ref ref="CONSOLE2"/>

  </root>

  <!-- 在SpringBoot中，可以通过springProfile属性来实现在不同环境上执行不同的输出策略，如下示例中指定prod和dev上有不同的策略-->
  <!-- 指定在prod环境使用的输出配置-->
  <springProfile name="prod">
    <root level="info">
      <appender-ref ref="FILE_ERROR"/>
    </root>
  </springProfile>

  <!-- 指定在dev环境使用的输出配置-->
  <springProfile name="dev">
    <root level="info">
      <appender-ref ref="FILE_INFO"/>
      <appender-ref ref="FILE_ERROR"/>
    </root>
  </springProfile>
</configuration>


```

上面的示例xml中引用的一个application.yml中的属性定义如下：

![](\assets/post_pics/2020-01-02-springboot_logback_notes.md/md_pics_2020-01-02-16-08-00.png)


### 3 LogBack与Log4j的区别

`LogBack`是基于`SLF4j(Simple Logging Facade for Java)`实现的日志框架，各方面性能相比`Log4j`要更优一些。

`LogBack`和`Log4j`是非常相似的，`LogBack`的内核重写了，在一些关键执行路径上性能提升10倍以上。而且`LogBack`不仅性能提升了，初始化内存加载也更小。

在使用`LogBack`的时候，不需要在代码中或配置文件中指定你打算使用哪个具体的日志系统，因为`SLF4J`提供了统一的记录日志的接口，只要按照其提供的方法记录即可，最终日志的格式、记录级别、输出方式等通过具体日志系统的配置来实现，因此**可以在应用中灵活切换日志系统**。

**推荐使用`SLF4j/LogBack`替换`Log4j`作为日志打印框架**。



---

欢迎关注我的公众号“**架构笔录**”，原创技术文章第一时间推送，也可互动一起探讨交流技术。

<center>

   ![](https://raw.githubusercontent.com/veezean/pic_assets/master/assets/comm_pics/contact/gongzhonghao.png)

</center>