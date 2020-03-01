---
layout: post
title: "Android TV开发中实现遥控器上下按的时候焦点在列表中循环滚动"
date: 2019-07-24
categories: 安卓TV
tags: AndroidTV RecyclerView 焦点处理
column: 过程记录
---

* content
{:toc}


在Android TV项目开发中遇到个诉求，即要求用户按遥控器的上键或者下键的时候，焦点可以在列表中循环滚动，记录下对应的试下你过程。





## 诉求分析

其实这个诉求，在电视使用的时候算是个比较常见的功能了。用户在左侧选择栏目的时候，当按下键到最后一个的时候，继续按下键，则焦点跳转到列表的第一个项目上。

实现的时候，也是遵循这个思路。监听按键事件，并做对应的响应处理：

> 如果按下下键，当前在列表最后一个，则指定焦点跳转到列表第一个item上。
> 如果按下上键，当前在列表第一个，则指定焦点跳转到列表最后一个item上。

## 代码实现

按照上面的分析，实现对应的代码（示例代码，假设列表中一共只有4个item）：

```java
viewHolder.itemView.setOnKeyListener(new View.OnKeyListener() {
    @Override
    public boolean onKey(View view, int i, KeyEvent keyEvent) {
        //此处keyevent分为按下和按键松开两个动作，需要判断下是按键按下的时候就在首个位置，才需要滚到最后位置
        if (keyEvent.getAction() == KeyEvent.ACTION_DOWN && position == 0 && i == KeyEvent.KEYCODE_DPAD_UP) {
            //TODO 优化写法
                Message msg = Message.obtain();
                msg.what = 1;
                msg.arg1 = 3; // 当前为0，按上键调到最下面一个
                handler.sendMessage(msg);
                return true;
        }
        if (keyEvent.getAction() == KeyEvent.ACTION_DOWN && position == 3 && i == KeyEvent.KEYCODE_DPAD_DOWN) {
            //TODO 优化写法
            Message msg = Message.obtain();
            msg.what = 1;
            msg.arg1 = 0; // 当前为3，按上键调到最上面一个
            handler.sendMessage(msg);
            return true;
        }
        return false;
    }
});
```

此处需要注意的一个问题就是`keyEvent.getAction() == KeyEvent.ACTION_DOWN`判断是 **必须** 的。因为按下一个按键会有两个事件，分别是按下前和按下后，此处必须要判断下在按下前进行处理。举个例子：

> 当前在第2个item上，当遥控器摁下`向上`按钮时，当动作完成后，焦点会跑到**第一个Item**上，而此时又会发送一个KeyEvent过来。如果没有判断KeyEvent的话，就会**导致焦点刚跑到第一个上面就即刻飞到最后一个Item上面**去了。

## 实现效果

看一下最终实现后的效果图：

![](/assets/post_pics/2019-07-24-focus%20loop%20move%20in%20the%20recyclerview.md/problem_pics5.gif)

---

欢迎关注我的公众号“**架构笔录**”，原创技术文章第一时间推送，也可互动一起探讨交流技术。

![](https://raw.githubusercontent.com/veezean/pic_assets/master/assets/comm_pics/contact/gongzhonghao.png)
