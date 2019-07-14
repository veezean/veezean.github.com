---
layout: post
title: "谈谈HashCode与HashMap相关概念"
date: 2017-10-20
categories: JAVA
tags: JAVA HashCode HashMap
---

* content
{:toc}

记录下JAVA中hashcode的相关概念，以及HashCode对于hash类容器的重要性，以及常见的一些hash冲突与冲突后的解决方案等。



## HashCode

hashCode,即一个Object的散列码。

HashCode的作用：

1. 对于List、数组等集合而言，HashCode用途不大；
2. 对于HashMap\HashTable\HashSet等集合而言，HashCode有很重要的价值。

HashCode在上述HashMap等容器中主要是用于寻域，即寻找某个对象在集合中的区域位置，用于提升查询效率。

一个对象势必会存在多个属性字段，而选择什么属性来计算hashCode值，具有一定的考验。因为如果选择的字段太多，而HashCode()在程序执行中调用的非常频繁，势必会影响计算性能；如果选择的太少，计算出来的HashCode势必很容易就会出现重复了。

### hashcode与equals

对于Object对象而言，其默认提供的equals方法实现就是简单的return this == obj;即只有两个对象都指向了同一个内存地址对象的时候，才会return true。  对于需要比较内容是否相等的情况时，必须要自行覆写equals方法。

对于覆写equals方法的时候，通常要求必须同时覆写HashCode()方法。hashCode即对象的散列码，int类型。

约定：

1. 覆写equals方法时必须覆写hashcode()
2. 如果两个对象的equals方法返回值为true，则两个object对应的HashCode值应该相同
3. 如果**两个对象的equals方法返回false，但是这两个object对应的HashCode是有可能会相同的**，（但是必须意识到： HashCode返回独一无二的值，对后续存储此对象的hashtable\hashmap等容器更好的处理有很大帮助。因此覆写的时候可以考虑下hashcode的算法，尽量避免重复情况）

### 重要的细节

在《Java编程思想》一书中的一段话：

> “设计hashCode()时最重要的因素就是：无论何时，对同一个对象调用hashCode()都应该产生同样的值。如果在讲一个对象用put()添加进HashMap时产生一个hashCdoe值，而用get()取出时却产生了另一个hashCode值，那么就无法获取该对象了。所以如果你的hashCode方法依赖于对象中易变的数据，用户就要当心了，因为此数据发生变化时，hashCode()方法就会生成一个不同的散列码”。

这个对编码的实际指导意义就是，**在使用类似HashMap等容器的时候，其Key值尽量避免使用可变对象**；或者说如果Key值使用了可变变量，务必需要保证此对象作为key值放入map中的时候，与其后续从map中查找的时候，其值不会被改变；换言之，就是如果使用某个对象作为Key值，则此对象的equals和HashCode实现的时候，尽量避免使用易变字段，否则如果对象中属性值变更之后，再去get（key）的时候，就会获取不到。

> hashCode实际应用中存在重复的可能性，因此实际编码中为了节省空间而使用HashCode作为HashCode的key值进行存储的做法是不正确的。


## HashMap & HashTable

讲HashMap之前，先说下数组与链表。

1. 数组：

数组存储区间是连续的，占用内存严重，故空间复杂的很大。但数组的二分查找时间复杂度小，为O(1)；数组的特点是：寻址容易，插入和删除困难；

2. 链表：

链表存储区间离散，占用内存比较宽松，故空间复杂度很小，但时间复杂度很大，达O（N）。链表的特点是：寻址困难，插入和删除容易。

为了综合上述两者的优点，便引申出了本次要重点学习的哈希表。

哈希表的优势：易于寻址、插入删除也容易，且不占太多内存。

### hashtable
哈希表是由数组+链表组成的，一个长度为16的数组中，每个元素存储的是一个链表的头结点。那么这些元素是按照什么样的规则存储到数组中呢。一般情况是通过hash(key)%len获得，也就是元素的key的哈希值对数组长度取模得到。比如上述哈希表中，12%16=12,28%16=12,108%16=12,140%16=12。所以12、28、108以及140都存储在数组下标为12的位置。

### hashmap

[http://blog.csdn.net/hxpjava1/article/details/55670439](http://blog.csdn.net/hxpjava1/article/details/55670439)

### hashcode 在 HashMap中的作用

关于HashMap的较为深入的介绍，参见：

[http://www.cnblogs.com/chenssy/p/3521565.html](http://www.cnblogs.com/chenssy/p/3521565.html)

[http://blog.csdn.net/hxpjava1/article/details/55670439](http://blog.csdn.net/hxpjava1/article/details/55670439)


### HashMap对于hashCode重复场景的容错处理

一般情况下，hashcode重复的可能性还是比较高的（尤其是使用eclipse默认生产HashCode的算法的时候），即使是某些自行定义的hash算法，也无法保证数据量足够大的时候不会出现HashCode重复，那么诸如hashMap之类的容器对象是如何实现HashCode重复的容错处理的呢？

[http://www.cnblogs.com/dolphin0520/archive/2012/09/28/2700000.html](http://www.cnblogs.com/dolphin0520/archive/2012/09/28/2700000.html)

解决冲突主要有两种方式。

1. 开放地址法：

即当一个关键字和另一个关键字发生冲突时，使用某种探测技术在Hash表中形成一个探测序列，然后沿着这个探测序列依次查找下去，当碰到一个空的单元时，则插入其中。比较常用的探测方法有线性探测法，比如有一组关键字{12，13，25，23，38，34，6，84，91}，Hash表长为14，Hash函数为address(key)=key%11，当插入12，13，25时可以直接插入，而当插入23时，地址1被占用了，因此沿着地址1依次往下探测(探测步长可以根据情况而定)，直到探测到地址4，发现为空，则将23插入其中。

2. 链地址法：

采用数组和链表相结合的办法，将Hash地址相同的记录存储在一张线性表中，而每张表的表头的序号即为计算得到的Hash地址。如下所示：

    HashCode1 --  [E1--> E2 --> E3]
    HashCode2 --  [E4--> E5 --> E6 --> E7]

3. 再哈希法：

当发生冲突时，使用第二个、第三个、哈希函数计算地址，直到无冲突时。缺点：计算时间增加。