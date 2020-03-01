---
layout: post
title: AndroidStudio中各种常用操作快捷键记录
date: 2017-11-30
categories: 工具使用
tags: AndroidStudio 快捷键
---

* content
{:toc}

介绍一些常用的AS快捷键。虽然也可以直接设置AS的快捷键与eclipse相同，想想后续主用AS，还是学习下AS自身的快捷键更好。



### 常用的AS的默认快捷键

---

> 1. **ctrl + N**                根据类名查找JAVA类， 类似于eclipse中的Ctrl+Shift+T
> 2. **ctrl + Shift + N**        根据文件名查找所有的文件， 类似于eclipse中的Ctrl+Shift+R
> 3. **ctrl + alt + L**          格式化代码，类似于eclipse中的ctrl + shift +F
> 4. **ctrl + alt + O**          自动优化import的包与类，类似于eclipse中的ctrl + O
> 5. **alt + insert**            自动生成代码（如getter\setter\toString\equals等），类似于eclipse中的shift+alt+s
> 6. **alt + shift + c**         最近的改动点（对比最近修改的代码）
> 7. **ctrl + E**                最近改动过的文件列表
> 8. **ctrl + f**                字符串查找
> 9. **ctrl + r**                字符串替换
> 10. **ctrl + alt + space**      类名与接口名提示
> 11. **ctrl + shift + space**    自动补全代码（与上面的有啥区别？）类似eclipse中的alt + /（**自己定义为shift + space**）
> 12. **ctrl + J**                自动生成对应的代码片段（比如foreach分支等， 或者根据预置的短语与模板自动生成代码）
> 13. **tab**                     代码标签模板输入后可以tab直接补全
> 14. **shift + F6**             重构，重命名，即批量替换选择的对象名称为新的名称，类似eclipse中的shift+alt+R
> 15. **ctrl + x**                删除行，类似于eclipse中的ctrl + d
> 16. **ctrl + d**                复制行，类似于eclipse中的ctrl + alt + ↓
> 17. **ctrl + H**                显示类的结构图（继承关系等层级关系展示出来）
> 18. **ctrl + Q**                显示方法或者类的注释信息，类似于eclipse中鼠标放到对应方法上自动浮现的javadoc信息（AS中可以在File--settings-Editor-General-Other中选择Show quick documentation on mouse move也可以实现鼠标悬浮提示）
> 19. **alt + F1**                左侧的树型导航中，跳到当前文件所在的位置（好像AS中左侧树光标不是在当前右侧打开的文件位置，不知道能不能设置，后续看下）
> 20. **alt + ←/→**               右侧打开的代码页签中，逐个切换
> 21. **alt + ctrl + ←/→**        跳到上一次浏览的位置,类似于eclipse中的alt + ←/→
> 22. **ctrl + shift + backspace**跳转到上一次编辑的地方
> 23. **alt + ↑/↓**               在一个代码类中，从当前位置跳到上一个或者下一个方法中
> 24. **ctrl + shift + ↑/↓**      代码片段向上或者向下移动，有个特点，就是选中的代码行只能在方法内部上下移动，无法跳出当前所在方法；如果光标放在方法定义行上，则会将整个方法整体移动到前一个方法前面或者后一个方法后面。
> 25. **ctrl + F12**              显示当前类的结构（一个类中的几个公共方法、私有方法、变量信息等等outline信息， AS默认左侧有个Structure页签，将其拽到右侧并常显，即和eclipse中一样展示了。）
> 26. **ctrl + W**                重复按，会依次从小范围选择到大范围，先单词、再整行、再整个方法、再整个类，适合需要选中整个方法的场景
> 27. **ctrl + shift + insert**   可以选择剪切板中指定的内容进行粘贴，类似于ctrl + v的高级版本，可以选择最近几次copy过的内容进行粘贴
> 28. **alt + F7**                查找方法或者类的调用

---


更多可以参考 [http://www.android-studio.org/index.php/docs/experience/142-androidstudio-shortcut-keys](http://www.android-studio.org/index.php/docs/experience/142-androidstudio-shortcut-keys)

---

欢迎关注我的公众号“**架构笔录**”，原创技术文章第一时间推送，也可互动一起探讨交流技术。

<center>

   ![](https://raw.githubusercontent.com/veezean/pic_assets/master/assets/comm_pics/contact/gongzhonghao.png)

</center>