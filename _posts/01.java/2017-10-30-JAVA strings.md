---
layout: post
title: "记录下JAVA中的String的众多细节"
date: 2017-10-30
categories: JAVA
tags: JAVA String
---

* content
{:toc}

String, 是JAVA中常见的一个引用类型，且其具有一定的特殊性。String类型被设置为final型，即不可继承，也就不可修改其中的实现。本文主要是介绍了下String在JVM中一些细节层面的表现，以及使用过程中一些注意事项等。





## String可以改变吗

String被设置为final型的，通常情况下是不可以改变的。

但是，从源码中可以得知，其字符串存储的时候使用的是char[]，虽然被标识为final型，但是可以通过反射等方式修改其中的值，但是不推荐。

反射 修改字符串实际值的步骤 ：
> 1. 获反射获取到数组对应的字段Field
> 2. 修改器访问属性为可访问
> 3. 修改其中的数组中的值。

详细可以参考下[http://www.cnblogs.com/fguozhu/articles/2661055.html](http://www.cnblogs.com/fguozhu/articles/2661055.html)，讲解的非常详细。

## 关于JAVA虚拟机中的字符串常量池

在JAVA虚拟机（JVM）中存在着一个字符串池，其中保存着很多String对象，并且可以被共享使用，因此它提高了效率。由于String类是final的，它的值一经创建就不可改变，因此我们不用担心String对象共享而带来程序的混乱。字符串池由String类维护，我们可以调用intern()方法来访问字符串池。

以`String s = "abc"`为例，这行代码被执行的时候，**JAVA虚拟机首先在字符串池中查找是否已经存在了值为"abc"的这么一个对象，它的判断依据是String类equals(Object obj)方法的返回值。如果有，则不再创建新的对象，直接返回已存在对象的引用；如果没有，则先创建这个对象，然后把它加入到字符串池中，再将它的引用返回**。

对于字符串是否会进入常量池中：
> 1. 如果是使用字面量的方式（即双引号定义字符串）定义字符串，JVM执行时会首先去常量池中查找是否已经存在，如果存在就直接返回，如果不存在就在常量池中创建，并返回引用。
> 2. 如果是使用new String（）的方式创建字符串，则必须手动调用intern()方法，才会去执行将内容放入常量池的操作（这个JDK6、7、8有着不同的实现逻辑，后面详细介绍。）


### 常量池的内存模型介绍

关于详细介绍，看下下面几个介绍，讲解的非常详细，也很透彻：

[http://www.jianshu.com/p/4ee6aec39c89?from=groupmessage](http://www.jianshu.com/p/4ee6aec39c89?from=groupmessage)

[http://blog.csdn.net/zhushuai1221/article/details/52663818](http://blog.csdn.net/zhushuai1221/article/details/52663818)

[http://blog.csdn.net/baidu_31657889/article/details/52315902](http://blog.csdn.net/baidu_31657889/article/details/52315902)


要点阐述如下：

1. JDK6版本， 常量池放在Perm方法区，与Heap区完全独立。
2. JDK7或者JDK8版本，常量池位于Heap区。



## 字符串创建与存储机制

1. 对于`String s = "aaa" `直接双引号的方式：

这种情况下， 首先会到常量池中查找下是否已经存在相同内容的字符串，如果有的话，则直接将变量s指向常量池中已经存在的字符串上面。

2. 对于`String s = new String("xxx")`的方式：

这种情况下， 首先会判断下倡廉吃中是否有相同值得字符串，如有，则拷贝一份到Heap中，然后返回heap中的地址；如果常量池中不存在，则会直接在heap中创建一份，然后将heap地址返回。

此处注意一点，就是如果"xxx"不存在与常量池中的话，也会先在常量池中创建一个（因为参数也是个字符串，下面章节中有讲）。

### 关于String的intern()方法

讲完上面的创建存储机制，不得不提及一个不常用的`intern()`方法。这个方法的作用是，**在常量池中查找是否有与当前字符串值相同的常量，如果没有，则新建常量并返回常量引用，但当前引用不变；如果有直接返回引用**。`intern()`方法的设计初衷就是重用String对象，进而可以节省内存消耗。

先看下下面这个代码：
```java
    String str1 = new String("SEU")+ new String("Calvin");  
    System.out.println(str1.intern() == str1);   
    System.out.println(str1 == "SEUCalvin");  
```
执行结果：
```
    true
    true
```

而下面的代码：
```java 
    String str2 = "SEUCalvin";//新加的一行代码，其余不变  
    String str1 = new String("SEU")+ new String("Calvin");  
    System.out.println(str1.intern() == str1);   
    System.out.println(str1 == "SEUCalvin");  
```
执行结果：
```
    false
    false
```

上面两段代码，看似str2与其余的没有任何关系，却影响到了str1的输出结果。这个实际上与intern()方法有关系。


**intern()方法在JDK1.6中的作用是：**比如`String s = new String("SEU_Calvin")`，再调用`s.intern()`，此时返回值还是字符串"SEU_Calvin"，表面上看起来好像这个方法没什么用处。但实际上，在JDK1.6中它做了个小动作：检查字符串池里是否存在"SEU_Calvin"这么一个字符串，如果存在，就返回池里的字符串；如果不存在，该方法会把"SEU_Calvin"添加到字符串池中，然后再返回它的引用。

**intern()方法在JDK1.7或者1.8版本中的作用是：**比如`String s3 = new String("1") + newString("1")`，此时s3对象对应的内存内容`“11”`存储在Heap区域，此时常量池中并无字符串11， 执行`intern()`方法的时候，先去常量池中看下`11`是否已经存在，如果不存在的话则直接将引用指向Heap中`"11"`字符串的地址。（这个地方是JDK7以上版本与JDK6的差别所在，即JDK6如果发现常量池中不存在，则会在常量池中创建一份，而JDK7或者JDK8中，如果常量池中没有，则直接将引用指向Heap中已有的字符串对象，而不会重新创建， 因此`s3 == s3.intern()`，而这个在JDK6中是false的）

## 差异点比较

### String使用+直接拼接

这种情况需要分两种情况来讨论：
1、 都是确定的字符串常量之间进行的+号拼接的时候，由于在编译器就可以确定其具体值了，所以编译器在编译期的时候就会把这些常量拼接的字符串解析为一个整的字符串常量。举例如下：

```java
    String s1="helloworld"; 
    String s2="hello" + "word"; 
    System.out.println( s1==s2 ); 
```
执行结果：
```
    true （可以看出s0跟s1是同一个对象，因为编译期的时候s2就已经被解析为helloworld了） 
```

2、 不确定的变量之间使用+拼接的时候，由于编译器无法在编译期进行优化，只能在运行期动态分配处理。举例如下：

```java
    String s1  =  "a"; 
    String s2  =  "b"; 
    String s3  =  "c"; 
    String s4  =   s1  +  s2  +  s3;
    String s5  =   "a" + "b" + "c";
    System.out.println( s4==s5 ); 
```
    执行结果：
```
    false  （因为s5编译期就确定为一个常量，而s4运行期动态分配）
```

    s4的JDK层实现逻辑如下：
	
```java
    StringBuffer temp = new StringBuffer();
    temp.append(s1).append(s2).append(s3);
    String s = temp.toString();
```

由JDK层的实现代码可以看出，每执行一次+操作，就会生成一个StringBuffer对象，然后生成一个新的字符串对象。因此，如果是for循环中对多个字符串进行拼接的时候，可以直接使用StirngBuffer或者StringBuilder进行append操作，可以减少N-1次StringBuffer对象的创建。

3、 final修饰的字符串变量，在编译的时候也会被当做常量进行优化，如下所示：
```java
    String s0 = "ab"; 
    final String s1 = "b"; 
    String s2 = "a" + s1;  
    String s3 = "b";
    String s4 = "a" + s3;
    System.out.println((s0 == s2));
    System.out.println((s0 == s4)); 
```

执行结果：
```
    true
    false
```

因为s1是final型的，所以编译期会被当做确定常量，s2直接编译期被优化为ab，而s3只是普通变量，无法编译期优化，所以按照字符串拼接的方式，会生成新的字符串对象。

### 关于String不同操作过程生成几个对象的问题

关键点：对于字符串字面量的形式（即双引号的方式），只有当字符串常量池中不存在相同的字符串的时候才会创建；对于new的方式一定会有新对象产生。

1、 `String s = "abc"`创建了几个对象？

这个语句创建了一个对象或者0个对象，取决于abc这个字符串是否已经在常量池中存在。

2、 `String s = new String("abc")`创建了几个对象？
这个语句等价于下面的语句：

```java
    String param = "abc";
    String s = new String(param);
```

因此，这个过程一共创建了两个对象或者一个对象，即param与s（取决于常量池中是否已经有abc字符串了）。

3、 `String s1 = "abc"; String s2 = "abc"`创建了几个对象？

这个情况也是只创建了一个对象或者0个对象"abc"，然后s1和s2都只是指向同一个"abc"内存地址的引用，如果"abc"已经在常量池中的时候，就不会新创建。

4、 `String s = "a" + "b"`创建了几个对象？

这个实际上与`String s = "ab"`是等价的，这个过程新创建了1个对象或者0个对象。

5、 `String c = "xx" + "yy " + a + "zz" + "mm" + b`创建几个对象？
这个语句编译优化后等价于下面的逻辑：

```java
    String c = (new StringBuilder().append("xxyy").append(a).append("zz").append("mm").append(b)).toString();
```

即：如果+进行拼接的时候，从左往右，会将最初的几个常量都直接合并为1个常量（比如xx和yy），然后与变量进行拼接，然后出现在变量后面的常量就没法再合并到一起了（比如zz和mm），只能逐个累加。
此过程中，会产生1个StringBuilder对象和一个String对象。


### 关于`==`与`equals`以及`HashCode`

简而言之，==用于比较简单的数据类型，或者看下某两个对象引用是否指向堆内存中的同一块地址；equals用于比较两个对象的内容是否相同。

对于Object对象而言，其默认提供的equals方法实现就是简单的`return this == obj;`即只有两个对象都指向了同一个内存地址对象的时候，才会return true。  对于需要比较内容是否相等的情况时，必须要自行覆写equals方法。

对于覆写equals方法的时候，通常要求必须同时覆写HashCode()方法。hashCode即对象的散列码，int类型。

约定：

1. 覆写equals方法时必须覆写hashcode()
2. 如果两个对象的equals方法返回值为true，则两个object对应的HashCode值应该相同
3. 如果**两个对象的equals方法返回false，但是这两个object对应的HashCode是有可能会相同的**，（但是必须意识到： HashCode返回独一无二的值，对后续存储此对象的hashtable\hashmap等容器更好的处理有很大帮助。因此覆写的时候可以考虑下hashcode的算法，尽量避免重复情况。 当然，就算出现重复，HashMap等容器也是有容错处理机制的，也是可以放入Map的，这个后续会深入分析。）


## 常见的几个String操作类的比较

1. StringBuilder

线程不安全，但是性能比较高。

2. StringBuffer

线程安全，但是性能比StringBuilder略差。因为其中的很多方法实现上都加了线程同步锁。

3. StringTokenizer

字符串分割处理工具类。


---

欢迎关注我的公众号“**架构笔录**”，原创技术文章第一时间推送，也可互动一起探讨交流技术。

![](https://raw.githubusercontent.com/veezean/pic_assets/master/assets/comm_pics/contact/gongzhonghao.png)
