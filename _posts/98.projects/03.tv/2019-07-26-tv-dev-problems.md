---
layout: post
title: "Android TV开发中以dp为单位布局存在的问题解决"
date: 2019-07-26
categories: 安卓TV
tags: AndroidTV 布局
column: 过程记录
---

* content
{:toc}

记录下Android TV开发中以dp为单位布局在不同机器显示效果的差异，以及对应的解决措施。





## 问题描述

最初的时候，右侧RecyclerView中每一个Item的高度是布局的时候指定的以dp为单位的固定值。整体预期的效果是右侧以`四宫格`的形式展示。

先来看APP下在模拟器上的显示效果：

![](/assets/post_pics/2019-07-26-tv-dev-problems.md/Screenshot_1564208596.png)

再看下将APP安装到真机上的效果：

![](/assets/post_pics/2019-07-26-tv-dev-problems.md/Screenshot_1564208837.png)

很明显，android设备的尺寸千差万别，对于RecylcerView这种滚动列表而言，用固定item高度的方式进行布局，很难在不同设备上完全兼容。对于RecyclerView常规的使用场景而言，上面的这种效果也无可厚非。但是我们的预期是右侧能展示成为`四宫格`，显然上图的效果是没法接受的。


## 解决方法

前面说了固定高度是不行的，而我们又必须要一个屏幕能显示出4个方块，还必须要兼容各种尺寸的设备。那就只能`运行时动态的去计算对应高度与宽度`了。

看下整个界面的布局示意图：

![](/assets/post_pics/2019-07-26-tv-dev-problems.md/md_pics_2019-07-27-15-10-01.png)

则每个方块的高度与宽度，便可以通过如下公式计算出来：

```
itemHeight = (LayoutHeight / 2) - (margin * 2)
itemWidth = (LayoutWidth / 2) - (margin * 2)
```

按上面的分析，在RecyclerView对应的Adapter类的`onBindViewHolder`方法中进行对应的处理计算，如下：

```java
        int marginPx = ScreenUtil.dip2px(rightFrameLayoutId.getContext(), 12);
        int heightPx = rightFrameLayoutId.getHeight() / 2 - marginPx * 2;
        int widthPx = rightFrameLayoutId.getWidth() / 2 -  marginPx * 2;

        FrameLayout.LayoutParams layoutParams = new FrameLayout.LayoutParams(widthPx, heightPx);
        layoutParams.setMargins(marginPx, marginPx, marginPx, marginPx);

        viewHolder.itemView.setLayoutParams(layoutParams);
```

## 修改后效果

按照上述方法修改代码后，重新编译APK包安装到电视真机上，效果如下：

![](/assets/post_pics/2019-07-26-tv-dev-problems.md/Screenshot_1564209331.png)

和模拟器上效果一致，达到预期效果。


---

欢迎关注我的公众号“**架构笔录**”，原创技术文章第一时间推送，也可互动一起探讨交流技术。

![](https://raw.githubusercontent.com/veezean/pic_assets/master/assets/comm_pics/contact/gongzhonghao.png)
