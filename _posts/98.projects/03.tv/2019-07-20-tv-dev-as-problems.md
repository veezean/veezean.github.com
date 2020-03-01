---
layout: post
title: "AndroidStudio编译时aapt报错定位方法"
date: 2019-07-20
categories: 安卓TV
tags: AndroidStudio aapt
column: 过程记录
---

* content
{:toc}

记录下android开发过程中遇到的Android Studio中编译时报aapt错误的解决过程。






## 报错信息

```
 com.android.ide.common.process.ProcessException: org.gradle.process.internal.ExecException: Process
'command 'D:\workspace\android_dev\sdk\build-tools\28.0.3\aapt.exe'' finished with non-zero exit value 1
```

但是AS中提供的定位信息太少，因此需要获取到更多的信息。

## 定位方法

在AS的terminal窗口中，执行下面的命令：

```
gradlew processDebugResources --debug
```

可以找到日志如下：

```
18:37:20.929 [DEBUG] [org.gradle.process.internal.ExecHandleRunner] waiting until streams are handled...
18:37:22.228 [DEBUG] [org.gradle.process.internal.DefaultExecHandle] Changing state to: FAILED
18:37:22.230 [DEBUG] [org.gradle.process.internal.DefaultExecHandle] Process 'command 'D:\workspace\android_dev\sdk\build-tools\28.0.3\aapt.exe'' finished with
 exit value 1 (state: FAILED)              
18:37:22.279 [INFO] [org.gradle.api.Project] Failed to generate resource table for split ''
18:37:22.328 [ERROR] [org.gradle.api.Project] D:\workspace\code\emtv\git\emtv\app\src\main\res\values\colors.xml:12:5-59: AAPT: Color value not valid -- must b
e #rgb, #argb, #rrggbb, or #aarrggbb (at 'half_transparency_black' with value '#e000000').

18:37:22.329 [DEBUG] [org.gradle.api.internal.tasks.execution.ExecuteAtMostOnceTaskExecuter] Finished executing task ':app:processDebugResources'
18:37:22.330 [LIFECYCLE] [class org.gradle.TaskExecutionLogger] :app:processDebugResources FAILED

```

根据上面的信息，就知道resource文件具体错在哪了，对照log进行修改即可。




---

欢迎关注我的公众号“**架构笔录**”，原创技术文章第一时间推送，也可互动一起探讨交流技术。

![](https://raw.githubusercontent.com/veezean/pic_assets/master/assets/comm_pics/contact/gongzhonghao.png)