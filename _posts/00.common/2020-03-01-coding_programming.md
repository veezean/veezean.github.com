---
layout: post
title: "编程范式知多少——厘清主流编程范式（编程思想）"
date: 2020-03-01
categories: 编程理论
tags: 编程思想 编程范式 函数式编程 命令式编程 声明式编程 面向对象编程 响应式编程
---

* content
{:toc}

本文主要是对业界主流的`编程范式（编程思想）`做一个汇总阐述，厘清各个编程范式之间的差异点、优缺点等，希望能对大家系统的了解编程范式提供点帮助。






## 编程范式哪家强

### 3种主流编程范式
#### 命令式编程

看个例子：

> 周末，中午我想吃个烤鸡翅，然后我：
> 
> 去菜场买几个鸡翅；鸡翅洗净、腌制；放入烤箱，设定烘烤温度、时间，开始烤；烤箱中取出鸡翅，放入盘中。
> 
> 开始吃鸡翅...

例子中的`我`诉求很明显，要吃鸡翅，为了实现这个目的，然后具体的一步一步的去做对应的事情，最终实现了目的，吃上了鸡翅。——这就是`命令式编程`。

`命令式编程`的主要思想是关注计算机执行的步骤，即**一步一步告诉计算机先做什么再做什么**。命令式编程是最常规的一种编程方式，各种主流编程语言如C、C++、JAVA等都可以遵循这种方式去写代码。

举个例子，从给定的一个数字列表collection里面，找到所有大于5的元素，用JAVA语言可以如下方式来写：

```java {.line-numbers}
List<Integer> results = new ArrayList<>();
for (int num : collection) {
    if (num > 5) {
        results.add(num);
    }
}
```

这就是一个典型的`命令式编程`的写法，上面的执行过程如下：
> 1. 先定义一个result列表，用于存放执行的结果；
> 2. 对原始列表的元素进行逐个的遍历操作；
> 3. 对每个元素，都判断下是否大于5，如果大于则放到result列表里面去；
> 4. 继续for循环，直到遍历结束，执行完成。

这个代码明确的告诉了计算机具体的每一步的执行过程，然后计算机按照代码执行的流程逐个命令去执行就可以了。

通过上面的例子可以看出：
> `命令式编程`是通过`赋值`（由表达式和变量组成）以及`控制结构`（如，条件分支、循环语句等）来修改可修改的变量。

说白了，就是计算机的一个个指令序列，按照写定的顺序或者跳转逻辑（for/while/goto等）进行逐个执行。程序执行的效率，取决于指令序列的多少、变量占用的空间大小等因素决定。因此，便有了有衡量算法优劣的`时间复杂度`、`空间复杂度`的概念。


#### 声明式编程

继续前面吃鸡翅的例子。

> 周末，晚上我还想吃个烤鸡翅...
> 我：妈，我要吃烤鸡翅...
> （一段时间后）
> 妈：烤鸡翅来咯...
> 我开始吃鸡翅...

这个例子里，`我`想吃烤鸡翅，但是我没有像前面一样自己去动手一步步的做，而是将我的这个诉求传递给`妈`，然后`妈`将鸡翅给准备好，实现了`我`的诉求。对例中的`我`而言，只需要声明要吃鸡翅就行了，至于这个鸡翅是怎么做出来的，完全不用关心。——这就是`声明式编程`。


`声明式编程`的主要思想是**告诉计算机应该做什么，但不指定具体要怎么做**。典型的声明式编程语言，比如：SQL语言、正则表达式等。

```sql
SELECT * FROM collection WHERE num > 5
```

再同样用JAVA举个声明式编程的例子：

```java
List<Integer> results = getAllGreaterThan5Numbers(collection);
```

上面就是声明式编程的一个典型示例，对于当前调用方而言，只需要告诉计算机我现在需要从`collection`里面获取所有大于5的数字就行了，至于`getAllGreaterThan5Numbers()`这个方法具体是如何一步步执行将大于5的数字过滤出来的，调用方完全不用关心。


#### 函数式编程

先要搞清楚，**此函数非彼函数**。

对于程序员而言，所谓的`函数`，一般都是指的各个编程语言中的`函数/方法`，但是`函数式编程`中的`函数`，指的是**数学意义上的函数**，也即`映射关系`。

举个例子： 
```
y = f(x)
f(x) = x + 1
```

这个数学公式里，`f(x)`是一个`函数`，其实表名的也就是`x`和`y`之间的某种关系,也即`x`加上`1`之后就是`y`了。在这个例子里：
```
若x=1,则y=f(x)计算后的值为2。
若x=2,则y=f(x)计算后的值为3。
若x=3,则y=f(x)计算后的值为4。
...
```
所以说，函数的特点很明显了，**接收一个或多个输入，生成一个或多个对应的结果，且输入和输出之间的关系很明确且固定、不会受之前其它输入的影响**。也即：

> 当一个函数无论何时、无论谁调用、无论传入什么，只要保持相同的输入，不管调用多少次，都能够得到同样的结果（无副作用、引用透明性），这样，就可以说这个函数具备了`函数式`的特征。

了解了`函数式`的概念，再回头看下`函数式编程`。其`核心思想`与声明式编程是一致的，即：**只关注做什么而不是怎么做。所以这种层面上来说，函数式编程是声明式编程的一部分**。

```java
List<Integer> results = 
    collection.stream()
              .filter(n -> n > 5)
              .collect(Collectors.toList());
```

再回头看下前面关于函数的描述，在数学中，函数`f(x)`的输入变量`x`，其实就是代表一个需要带入到函数公式中计算的`具体的数值`。同理、类比到函数式编程场景中，函数中传入的变量也是不可赋值的（`final`类型的），即：**变量的值是不可变的**。具体而言：
- 函数传入的变量值，一旦绑定后就不能再发生变化了。
- 与其说传入的变量是值，不如说`传入的变量是个表达式`（惰性求值）。

当然咯，函数式编程的概念是个很庞大的理论体系，除了上面提及的几个特征，还有一些别的特性（比如`柯里化`），此处不做展开细述。

### 巅峰对决

#### 命令式编程 VS 声明式编程

老规矩，先说个生活中的例子：

> 一天下午，在家。
> 老婆：“老公，我要吃千层蛋糕”。
> ...(一段时间后)...
> 老公：“老婆，给你千层蛋糕”

上面就是个典型的`声明式编程`的例子，即只说要什么，不关注怎么做。`老婆`只告诉`老公`她要吃千层蛋糕，然后`老公`负责将千层蛋糕给到`老婆`，至于`老公`是怎么弄来这个千层蛋糕的，到底是出门买的、还是点的外卖、还是自己亲手做的，`老婆`完全不关心。

演变下上面的例子：

> 一天下午，在家。
> 老婆：“老公，我要吃千层蛋糕，你先去手机上下载个XX美食APP，然后搜索下需要哪些材料，然后步行去小区斜对面的菜场里面买一下鸡腿，再去对面超市里面买其余的辅料，最后回来做千层蛋糕给我吃”。
> ...(老公遵照老婆的指示，先下载APP，再搜索需要的材料，再去菜场买菜、去超市买辅料，回家开始做蛋糕...一段时间后完成...)...
> 老公：“老婆，给你千层蛋糕”

这个例子中的`老婆`指令详细的告知了一步步要怎么做，这就是`命令式编程`的雏形。显然，与前面的声明式编程相比，本例子中的`老婆`显然要事无巨细的操心、麻烦了很多，而前面的例子中，则可以当一个甩手掌柜，发出诉求后，等着吃就行了~~

对上面例子再展开描述下：

> 一天下午，在家。
> Step1
> 老婆：“老公，我要吃千层蛋糕”。
> Step2
> 老公开始搜索食谱教程，准备食材，跟着食谱一步步的做蛋糕（累的满头大汗）...
> Step3
> 老公：“老婆，给你千层蛋糕”

这个例子中，对于`老婆`而言，上面已经说过了，属于`声明式编程`。对于Step2中的`老公`而言，就是完全的`命令式编程`了，他需要知道具体的每一步需要做什么事情，最后将蛋糕做出来。

从上面的3个例子中可以看出来，如果要最终实现一个诉求，肯定是需要有人发出诉求、有人要具体操心去最终实现，而`老婆`能否当甩手掌柜，取决于`老公`是否足够全能（当然，程序猿老公肯定都是全能型的了，不然怎么会有老婆呢...哈哈）。

对应到开发场景中，例子中的`老婆`便是业务模块，而`老公`便是各种封装的工具类、开源库函数等，如果有够厉害的库函数支撑把底层复杂的逻辑都实现了，那业务逻辑层就是负责发号施令就可以了。

我们回到代码层面，继续看下前面声明式编程中的提到的那个例子的完整代码：

```java
// 声明式编程、只告诉应该要做什么，不关心具体怎么做
List<Integer> results = getAllGreaterThan5Numbers(collection);
...

// 命令式编程、告诉计算机具体的每一步操作逻辑
private List<Integer> getAllGreaterThan5Numbers(List<Integer> collection) {
    List<Integer> results = new ArrayList<>();
    for (int num : collection) {
        if (num > 5) {
            results.add(num);
        }
    }
    return results;
}
```

`声明式编程`是一种**最为简单的方式**，但`声明式编程`通常都是需要依赖于`命令式编程`的基础上才能实现的。之所以能实现`声明式编程`，是因为相关的库函数、或者自定义的工具函数（比如这里例子中的`getAllGreaterThan5Numbers()`方法）将底层具体一步步执行的逻辑给封装了，而这些工具方法，最终还是需要基于`命令式编程`的方式来实现的（是不是像极了前面“老公”和“老婆”的例子？）。

当然，`命令式编程`与`声明式编程`并无优劣之分，对于程序员而言，`声明式编程`的风格自然是更好的，这是人的惰性决定的。而且，从代码调试的角度来看，`声明式编程`不好调试，甚至无从调试（比如调用其他库或者依赖包的API、且没有源码的时候）。此外，对于大型项目系统而言，`命令式编程`、`声明式编程`两种形式肯定是相互依存的。

所以，这也给我们一个很重要的启示：
- 项目中，对于公共的工具方法的封装，一定要保证**绝对准确、绝对可靠**！
- 技术选型的时候，选择的三方或者开源的框架或者工具库的时候，一定要尽可能选择迭代**成熟且稳定**的。
- 站在巨人的肩膀上，有现成的工具时，**避免闭门造车**，重复造轮子。

#### 命令式编程 VS 函数式编程

当前硬件发展速度日新月异，几乎所有的CPU都是支持多核运行，而囿于CPU单核执行的效率瓶颈，多核编程的概念越来越受到重视。

对于`命令式编程`而言，其执行的时候只能占据着单核CPU进行处理（注意：这里与传统意义上的多线程并发不是一个概念），如果需要实现多核编程，则需要对代码进行改动、然后依托多线程并发编程等方式去实现，复杂度与实现难度都会高很多。

依旧是前面`命令式编程`所使用的例子，从给定的数字列表中，找出所有大于5的元素，前面也已经看过其`命令式编程`的代码如下：

```java
List<Integer> results = new ArrayList<>();
for (int num : collection) {
    if (num > 5) {
        results.add(num);
    }
}
```

很明显，我们都知道，这段代码去执行的时候，一定是从上到下在一个单核单线程中执行。那假定给定的collection数字列表特别大，for循环中的逻辑又比较耗时，则这段代码执行下来肯定是要耗费很长的时间，应该如何优化呢？——很容易想到，多线程！

继续，针对上面的例子，将其改造为`多线程`并行处理逻辑，可以这样（有多种方式，此处只是举个简单的实现~）：
> 1. 搞个线程池，里面维护若干个执行线程，比如10个。
> 2. 封装个执行Thread线程，run()方法里面封装具体的逻辑。
> 3. 将collection内容10等分，分别分配给线程池里面的10个线程一起处理。
> 4. 建立一个线程安全的结果集容器，用于存放各个线程处理筛选出来的结果。
> 5. 等待所有线程处理完成，得到最终的结果集。

具体的代码就不贴了，总之，可以看出来，在命令式编程中，要实现多核编程，需要额外的很多复杂处理逻辑才能实现、还要关注各种`信号量`、`锁`的概念，稍有不慎，还容易出现严重问题——这也是`命令式编程`的一个硬伤，关于`并发编程`的内容足够撑起一本厚厚的教科书。

再来看下`函数式编程`，`函数式编程`似乎天生就是为了`多核编程`而生的,它打破了由于`循环体`的存在而导致的只能单核运行的束缚。`函数式编程`的出现给各种库函数开拓了一片新天地，可以提供各种更方便的API接口。比如结合`JAVA`中的`Stream`流，并行操作甚至可以一行代码就搞定了：

```java
List<Integer> results = 
    collection.parallelStream()
        .filter(n -> n > 5)
        .collect(Collectors.toList());
```

## 最后

一个大型系统中，各种编程范式一般都是相互集成、互为补充的。只能说`在恰当的地方使用恰当的方式完成恰当的事情`，具体这个度，需要根据实际情况进行把控。当然咯，`函数式编程`作为近年来各种编程语言的新宠，还是值得学习下的,可以有效的简化我们的代码逻辑、增强可读性，提升并行处理效率等。

## 参考内容

本文中部分观点或者内容参考自互联网，相关引用地址如下：

- [函数式编程与命令式编程](https://www.jianshu.com/p/eebb15f2c0d5)


---
---

欢迎关注我的公众号“**架构笔录**”，原创技术文章第一时间推送，也可互动一起探讨交流技术。

![](https://raw.githubusercontent.com/veezean/pic_assets/master/assets/comm_pics/contact/gongzhonghao.png)


