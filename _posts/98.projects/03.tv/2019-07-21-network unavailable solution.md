---
layout: post
title: "Android TV开发中对网络异常场景的UI交互处理"
date: 2019-07-20
categories: 安卓TV
tags: Android toast
column: 过程记录
---

* content
{:toc}

记录下android tv视频播放器开发过程中对网络异常场景的处理。





# 网络断开场景的处理

## 实现分析

本来想着搞一个独立的网络异常的activity页面显示，但是那样会打破TV应用比较关注的沉浸式的体验，而且从网络异常页面退回的话，还需要用户再去按返回键，增加用户的操作复杂度。**因此这种方案被放弃了**，但还是可以看下效果：

![](/assets/post_pics/2019-07-21-network%20unavailable%20solution.md/problem_pics10.gif)

其实对于列表数据加载的场景，网络断开的时候，直接Toast的方式提示一下，保持用户当前界面显示即可（当然，用户体验更好的一种实现，就是监听网络变更状态，等网络恢复之后，自动获取新的数据）。。

## 实现效果

滚动增量加载的过程中网络断开，提示效果如下：

![](/assets/post_pics/2019-07-21-network%20unavailable%20solution.md/problem_pics7.gif)


切换栏目的时候无网络，效果如下：

![](/assets/post_pics/2019-07-21-network%20unavailable%20solution.md/problem_pics9.gif)

---

欢迎关注我的公众号“**架构笔录**”，原创技术文章第一时间推送，也可互动一起探讨交流技术。
![](https://raw.githubusercontent.com/veezean/pic_assets/master/assets/comm_pics/contact/gongzhonghao.png)