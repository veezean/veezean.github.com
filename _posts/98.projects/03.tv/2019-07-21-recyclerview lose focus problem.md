---
layout: post
title: "Android TV中RecyclerView下拉加载时焦点丢失乱飞问题"
date: 2019-07-21
categories: 安卓TV
tags: AndroidTV RecyclerView 焦点处理
column: 过程记录
---

* content
{:toc}

在android tv项目中，需要实现遥控器下翻的时候能动态加载分页数据，分页数据加载后会出现焦点乱飞的现象，记录下解决过程。





## 问题现象

具体问题现象可以看下下面的演示图。右侧列表翻页加载的时候，焦点本来在右侧item上，然后分页加载数据之后，焦点飞到了左上角去了：

![](/assets/post_pics/2019-07-21-recyclerview%20lose%20focus%20problem.md/problem_pics4.gif)

## 分析过程

遥控器控制焦点往下滚动的时候，会寻找页面中下一个可获焦view，而此时页面数据重新刷新了，重新渲染页面之后，原先对应的position位置信息也被冲掉了，因此右侧寻找焦点失败之后，焦点自动从整个activity中去寻找了。

## 解决措施


在Adapter中，覆写`getItemId`方法：

```java
    @Override
    public long getItemId(int position) {
        //覆写此方法，解决滚动分页加载时焦点丢失问题
        return position;
    }
```

然后：

```java
// 设置唯一标识生效
adapter.setHasStableIds(true);

// 关闭动画效果
recyclerView.setAnimation(null);
```


## 实现效果

按照上面的描述修改了下代码，再次验证，问题解决。

![](/assets/post_pics/2019-07-21-recyclerview%20lose%20focus%20problem.md/problem_pics3.gif)

---

欢迎关注我的公众号“**架构笔录**”，原创技术文章第一时间推送，也可互动一起探讨交流技术。

![](https://raw.githubusercontent.com/veezean/pic_assets/master/assets/comm_pics/contact/gongzhonghao.png)