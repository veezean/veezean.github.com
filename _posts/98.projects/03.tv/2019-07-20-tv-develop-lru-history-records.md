---
layout: post
title: "实现LRU排序滚动覆盖最早记录的历史播放列表功能"
date: 2019-07-20
categories: 安卓TV
tags: Android LinkedHashMap
column: 过程记录
---

* content
{:toc}

TV播放器中，需要实现一个历史记录列表展示的界面。实现的方式有很多种，本次基于LinkedHashMap进行了简单的定制，以一种较为优雅的方式来实现，记录一下。





## 实现分析

一个完整的视频播放器的历史记录功能，至少应该具备下面几点要求：

> 1. 列表中的记录不会无限制增大
> 2. 按照最近打开的顺序从上到下排序展示

## 代码实现

继承LinkedHashMap，自定义一个Map，主要实现2点：

> 指定按照LRU进行排序
> 限制此map中最大数据量，可以按照LRU原则自动覆盖优先级最低的数据

定义代码示意如下：

```java
public class LRULinkedHashMap<K, V> extends LinkedHashMap<K, V> {
    private static final long serialVersionUID = 1287190405215174569L;
    private int maxEntries;

    public LRULinkedHashMap(int maxEntries) {
        super(16, 0.75f, true);
        this.maxEntries = maxEntries;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        // 当总大小超过最大限制值的时候，移除最早的数据
        return size() > maxEntries;
    }
}
```

因为常规遍历Map的时候，默认都是从前往后遍历数据。如果需要按照最近播放情况排列显示的话，就需要对Map进行逆序遍历输出：

```java
public void printMap() {
    ListIterator<Map.Entry<Long, TestBean>> listIterator 
        = new ArrayList<>(map.entrySet()).listIterator(map.size());
    while (listIterator.hasPrevious()) {
        System.out.println(listIterator.previous());
    }
}
```

实际测试效果如下：

```
-----初始的时候，插入1、2、3三个数据-----
3=TestBean{name='3', id=3}
2=TestBean{name='2', id=2}
1=TestBean{name='1', id=1}
-----执行一次从map中get ID为2的记录（2记录排到优先级最高位置）-----
2=TestBean{name='2', id=2}
3=TestBean{name='3', id=3}
1=TestBean{name='1', id=1}
-----再执行一次更新ID为3的记录（3记录排到优先级最高位置）-----
3=TestBean{name='33', id=3}
2=TestBean{name='2', id=2}
1=TestBean{name='1', id=1}
-----再执行一次更新ID为2的记录（2记录排到优先级最高位置）----
2=TestBean{name='2', id=2}
3=TestBean{name='33', id=3}
1=TestBean{name='1', id=1}
-----再插入4、5、6三条记录，总数超过预设的最大值5个，将优先级最低的1记录自动删除）-----
6=TestBean{name='6', id=6}
5=TestBean{name='5', id=5}
4=TestBean{name='4', id=4}
2=TestBean{name='2', id=2}
3=TestBean{name='33', id=3}
```

从上面的实测结果来看，是基本满足了播放历史功能的诉求了的。

## 实现效果

看下播放历史功能的一个实现效果：

![](/assets/post_pics/2019-07-20-tv-develop-lru-history-records.md/problem_pics11.gif)

---

欢迎关注我的公众号“**架构笔录**”，原创技术文章第一时间推送，也可互动一起探讨交流技术。

![](https://raw.githubusercontent.com/veezean/pic_assets/master/assets/comm_pics/contact/gongzhonghao.png)
