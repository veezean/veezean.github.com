---
layout: post
title: "安卓开发中RecyclerView中ViewHolder复用带来的界面错乱问题"
date: 2019-07-20
categories: 安卓TV
tags: Android RecyclerView ViewHolder
column: 过程记录
---

* content
{:toc}

最近接触了个Android TV项目的开发，使用到了RecyclerView，遇到了一个页面错乱的问题。作为一个界面开发小白，看了半天的源码才弄明白原来ViewHolder竟然是复用的。记录下整个问题现象与解决过程吧，积累下自己的前端开发经验。





## 问题现象

电视播放界面上，左侧类别选项切换的时候，右侧显示的内容跟着切换。但是从第一个类别切换到第二个类别的时候，发现右侧左上角第一个item上面残留了上一类别中此位置的的一个图标。而从第二个类别再切回第一个类别的时候，发现第二个类别中的背景图片又残留到了第一个类别中的item上了。

如下所示：

![](/assets/post_pics/2019-07-20-tv-develop-problems-collection.md/problem_pics.gif)


## 分析过程

刚开始的时候，只有一个内容：

```
先create一个，再bind一次
2019-07-20 16:01:30.340 10749-10749/com.eastmoney.emtv I/System.out: ----------108173637---------
2019-07-20 16:01:30.340 10749-10749/com.eastmoney.emtv I/System.out: 165443738 -------- 0-----108173637------
```

第二个栏目，共有三个页面，之前已经create一个了，所以本次直接bind已有的一个。另外两个需要自己create并bind

```
2019-07-20 16:01:53.947 10749-10749/com.eastmoney.emtv I/System.out: 165443738 -------- 0-----108173637------

2019-07-20 16:01:53.975 10749-10749/com.eastmoney.emtv I/System.out: ----------187401686---------
2019-07-20 16:01:53.975 10749-10749/com.eastmoney.emtv I/System.out: 165443738 -------- 1-----187401686------

2019-07-20 16:01:53.984 10749-10749/com.eastmoney.emtv I/System.out: ----------37989107---------
2019-07-20 16:01:53.984 10749-10749/com.eastmoney.emtv I/System.out: 165443738 -------- 2-----37989107------
```

## 初衷分析
按照上面的分析，当然可以在bind操作的时候，每次都恢复下默认值，但这个代码写出来的话，后面维护起来就会是个巨大无比的坑。挖坑的事情，显然不是一个合格程序猿该做的事情~

如上实现的初衷是想在一个列表中能够显示出不同类型的item显示项目，而此前也没接触过前端界面相关的开发技能，所以最初的时候想当然的这么写了。

实际上，回头分析下诉求，无非就是同一个列表中，需要为Item指定不同的布局，显示为不同的样子。
有没有一种方式，能够为不同的Item指定不同的layout呢？显然是有的。

## 换种思路实现

看下RecyclerView源码中的API描述，其中`Consider using id resources to uniquely identify item view types.`这句似乎看到了希望，是的，可以为item指定resource id，也即layout资源文件咯？

```java
/**
 * Return the view type of the item at <code>position</code> for the purposes
 * of view recycling.
 *
 * <p>The default implementation of this method returns 0, making the assumption of
 * a single view type for the adapter. Unlike ListView adapters, types need not
 * be contiguous. Consider using id resources to uniquely identify item view types.
 *
 * @param position position to query
 * @return integer value identifying the type of the view needed to represent the item at
 *                 <code>position</code>. Type codes need not be contiguous.
 */
public int getItemViewType(int position) {
    return 0;
}
```
尝试在代码中覆写此方法，根据Item对象中属性值确定类型，然后返回不同的layout resource ID值，如下：

```java
/**
 * 复写此方法，用于根据类型指定不同的item布局，实现一个列表中展示不同item效果
 *
 * @param position 列表中的位置
 * @return 对应的布局信息
 */
@Override
public int getItemViewType(int position) {
    VideoItemVO videoItemVO = mDataset.get(position);
    // 如果是特殊类型Item，返回对应的layout布局信息
    if (videoItemVO.isSpecialItem()) {
        return videoItemVO.getItemLayoutId();
    }
    return super.getItemViewType(position);
}
```

按照上面的描述修改了下代码，再次验证，问题解决。

![](/assets/post_pics/2019-07-20-tv-develop-problems-collection.md/problem_pics2.gif)




