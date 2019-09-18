---
layout: post
title: "实现两个RecyclerView联动，从右侧返回左侧的时候返回之前的焦点"
date: 2019-07-24
categories: 安卓TV
tags: AndroidTV RecyclerView 焦点处理
column: 过程记录
---

* content
{:toc}

电视上进行视频播放的时候，遥控器选中左侧的某个栏目，然后摁下右键在视频列表信息中进行移动，再按下遥控器左键返回类别中的时候，预期的是能够继续回到此前浏览的那个页签。在Android TV对应APP开发的时候为了实现这个效果费了不少的时间试探，下面记录下一种有效的实现方案。





## 问题现象

最初开发调测的时候发现，左右两个RecyclerView之间进行焦点切换，返回的时候，焦点随意找了个最近的左侧item，并不是预期的回到原来焦点位置。见下图演示：

![](/assets/post_pics/2019-07-24-focus%20in%20two%20recyclerview%20move.md/problem_pics6.gif)

## 解决方案

在代码中进行遥控器按键事件的监听并进行定制处理：

> 1. 当前焦点在左侧列表中，遥控器按下`向右`键时，记录下左侧当前的焦点位置；
> 2. 当左侧列表重新获取到焦点的时候，判断如果是`向左`的按键事件，则将当前焦点指定为上次记录下来的item的位置。

看下代码实现，首先是左侧列表失去焦点前先记录当前位置：

```java
viewHolder.itemView.setOnKeyListener(new View.OnKeyListener() {
    @Override
    public boolean onKey(View view, int i, KeyEvent keyEvent) {
        if (i == KeyEvent.KEYCODE_DPAD_LEFT) {
            // 类别上左按键，无操作
            return true;
        }
        if (keyEvent.getAction() == KeyEvent.ACTION_DOWN && i == KeyEvent.KEYCODE_DPAD_RIGHT) {
            //TODO 优化写法
            boolean hasFocus = viewHolder.itemView.hasFocus();
            if (hasFocus) {
                // 记录下当前类别栏已经失去焦点
                GlobalViewDataManager.getInstance().updateFocusInCategory(false);
                GlobalViewDataManager.getInstance().updateCurrentCategory(position);
            }
            return false; //不拦截，继续交由上层处理
        }
        return false;
    }
});
```

然后再在监听焦点的方法里面，判断下是否是失焦重获的场景，如果是则将焦点重新给上一次的位置：

```java
viewHolder.itemView.setOnFocusChangeListener(new View.OnFocusChangeListener() {
    @Override
    public void onFocusChange(View view, boolean b) {
        if (b) {
            if (!GlobalViewDataManager.getInstance().isFocusInCategory()) {
                // 失去焦点后重新获得焦点
                GlobalViewDataManager.getInstance().updateFocusInCategory(true);
                //返回之前的焦点
                //TODO 优化写法
                Message msg = Message.obtain();
                msg.what = 1;
                msg.arg1 = GlobalViewDataManager.getInstance().getCurrentCategory();
                handler.sendMessage(msg);
                // 直接return，从右侧重新回到之前的类别，无需重新加载
                return;
            }

            GlobalViewDataManager.getInstance().updateFocusInCategory(true);
            GlobalViewDataManager.getInstance().updateCurrentCategory(position);
            GlobalViewDataManager.getInstance().updateAppendingLoad(false);
            itemDataLoader.asyncLoadAll(position);
        }
    }
});
```

## 实现效果

最终实现效果如下图所示，达到了预期的效果。

![](/assets/post_pics/2019-07-24-focus%20in%20two%20recyclerview%20move.md/problem_pics8.gif)

