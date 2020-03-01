---
layout: post
title: JAVA中LinkedHashMap要点记录
date: 2017-09-28
categories: JAVA
tags: JAVA LinkedHashMap
---

* content
{:toc}

## 构造函数中参数说明

构造函数中可能出现的几个参数说明如下：
> 1、`initialCapacity`初始容量大小，使用无参构造方法时，此值默认是16
> 
> 2、`loadFactor`加载因子，使用无参构造方法时，此值默认是 0.75f
> 
> 3、`accessOrder`false：基于插入顺序；true：基于访问顺序 

其余的几个参数都好理解，关键是最后一个参数，是LinkedHashMap区别于其余类型的HashMap的一个关键特性所在。 对于`accessOrder`设置为true的时候，其依据LRU的原则，会按照最近使用次数进行排序，最近使用的会被排在后面，不使用的会被排在后面。



## 控制Map中key最大条数

可以通过覆写`removeEldestEntry`方法，来实现控制此Map的最大key条数，当Map中key数量达到指定的设定值的时候，就会按照指定的排序规则，将排在最前面的数据给删除掉，然后插入新的数据（覆写方法之后，在调用put方法的时候会自动调用，无需业务代码中显式调用），示例代码如下所示：

```java
public class FixedLengthLinkedHashMap<K, V> extends LinkedHashMap<K, V> {
    private static final long serialVersionUID = 1287190405215174569L;
    private int maxEntries;
      
    public FixedLengthLinkedHashMap(int maxEntries, boolean accessOrder) {
        super(16, 0.75f, accessOrder);
        this.maxEntries = maxEntries;
    }
    
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > maxEntries;
    }
}
```
    
**比较适合需要排序，或者需要限制Map缓存数量、或者是需要清理最早或者最不常用的数据的场景。**

## 注意点

LinkedHashMap是**非线程安全**的，多线程场景需要注意。

---

欢迎关注我的公众号“**架构笔录**”，原创技术文章第一时间推送，也可互动一起探讨交流技术。

![](https://raw.githubusercontent.com/veezean/pic_assets/master/assets/comm_pics/contact/gongzhonghao.png)