---
layout: post
title: "实现完整网页保存为图片的方法"
date: 2019-07-14
categories: 前端
tags: html PhantomJS chromedriver
---

* content
{:toc}

业务场景中，会存在某些场景需要将网页内容快照保存下来的场景。因为有些网页内容是联网异步获取的，所以爬虫保存html页面的方式无法保证后续数据与此前的一致性，因此将网页内容以图片保存下来，是一种简单而直接的思路。本文档即针对上述诉求的技术可行性进行论证， 并给出可行的技术实现手段。



## 整体阐述

按照前面提出的思路，一种简单的业务处理场景可以抽象为如下的模型：

![](/assets/post_pics/2019-07-15-html%20page%20to%20jpg.md/2.png)

主机服务器上部署一个服务， 从来源处获取到 url 信息， 然后请求此 url 内容并生成截图保存在文件服务器中， 可以在数据库中保存此图片与 url 的映射关系， 便于后续查找。

下面主要阐述下如何实现根据 url 生成其对应内容全量截图（图中蓝色部分）。本文中主要提供了2中可选的实现方案，分别是:
* 通过 PhantomJS 方式
* 通过Chrome headless 方式

需要说明的是，在**GitHub 上显示 PhantomJS 已经暂停维护**了。仅从URL截图这一个诉求来分析的话，已有版本是完全满足要求的、且实现上更简单。如果有更多方面的考量，可以**优选Chrome headless方案**。

## 网页截图技术方案

### 通过 PhantomJS 实现

PhantomJS是一个基于webkit的JavaScript API。作为一个免费且开源的工具，支持Windows/Linux/Mac等多平台上运行，且可以通过JAVA/Python/bat/sh等方式进行调用。

以Windows平台为例，PhantomJS提供了一个exe文件，可以通过在JAVA或者Python中进行简单的封装调用即可，下面对其用法进行简单介绍。
编写其js脚本如下：

```js
var page = require('webpage').create();
page.open(url, function(status) {
  console.log("Status: " + status);
  if(status === "success") {
    page.render(pic_name);
  }
  phantom.exit();
});
```

则在cmd窗口中，执行`phantomjs.exe screenshot.js`命令，则会将js中指定的url页面内容生成图片并保存在指定的位置。

在工程中调用PhantomJS的用法如下：
#### JAVA实现

JAVA工程中可以通过拼接命令并调用exe文件执行抓取操作来实现。考虑先准备一份js模板，然后代码中处理替换掉js模板中的url和pic_name字段，并调用`phantomjs.exe screenshot.js`命令完成图片抓取。

代码DEMO片段如下：

```java
/**
* 将url内容转换为png图片保存
 * @param url 目标url地址
 * @param pngSavePath 图片保存位置
 */
public static void convertHtml2Png(String url, String pngSavePath) {
	// 读取js模板
	String templateJsContent = FileUtil.readFile("Template.js", "utf-8");
	// 将js模板中的url和图片路径占位符全部替换为实际的
	String realJsContent = templateJsContent
			.replace("url", "'" + url + "'")
			.replace("png_name", "'" + pngSavePath + "'");
	String realJsTempPath = "./tmp/" + UUID.randomUUID().toString() + ".js";
	// js内容写入临时文件
	FileUtil.writeFile(realJsTempPath, realJsContent);

	// 拼接cmd命令并执行
	String cmd = "phantomjs.exe " + realJsTempPath;
	try {
		Runtime.getRuntime().exec(cmd);
	} catch (Exception e) {
		// ...
	}

	// 删除js临时文件
	new File(realJsTempPath).deleteOnExit();
}
```

> 此方案需要安装相关环境信息如下：
> - JDK
> - PhantomJS

#### Python实现

Python中结合selenium和PhantomJS可以轻松实现页面全图截取，代码DEMO演示如下：

```python
from selenium import webdriver
import os

driver = webdriver.PhantomJS()

urls = open("urls.txt") 
for url in urls:
    driver.get(url)
    driver.save_screenshot(str(hash(url)) + '.png')

driver.close()
```

> 此方案需要安装相关环境信息如下：
> - Python（含selenium库）
> - PhantomJS（.exe放到python安装目录script目录下）

### 通过Chrome headless模式实现

如前面所述，PhantomJS在根据url生成图片方面已经满足要求了，但是PhantomJS目前暂停更新了。且在高版本的python selenium中已经将PhantomJS标记为deprecated并推荐使用chrome headless方式来替代。

所谓headless模式，也即无UI模式，在不打开chrome浏览器窗口的情况下，在后台进行无界面处理。

下面介绍下在python中通过chrome headless进行url全图保存的实现方式。参考如下的DEMO代码片段：

```python
from selenium import webdriver
import os
import time
from selenium.webdriver.chrome.options import Options

chrome_options = Options()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--disable-gpu')
chrome_options.add_argument('--start-maximized')

urls = open("urls.txt") 
for url in urls:
    driver = webdriver.Chrome(executable_path='chromedriver.exe', chrome_options=chrome_options)
    driver.get(url)
    # Get the actual page dimensions using javascript
    width = driver.execute_script("return Math.max(document.body.scrollWidth,document.body.offsetWidth, document.documentElement.clientWidth, document.documentElement.scrollWidth, document.documentElement.offsetWidth);")	 
    height = driver.execute_script("return Math.max(document.body.scrollHeight, document.body.offsetHeight,document.documentElement.clientHeight, document.documentElement.scrollHeight, document.documentElement.offsetHeight);")
    #resize
    driver.set_window_size(width,height)
    time.sleep(1)
    driver.save_screenshot(str(hash(url)) + '.png')
    driver.quit()
```

因为通过chrome headless的方式进行截图的时候，默认只截取当前显示屏幕区域。因此如果需要截取网页全部内容，便需要进行额外的处理（如上述代码中红色标识的代码片段）。在python中通过执行js语句，计算出网页真实的width和height值，然后对页面resize操作使其展示全部大小，之后再进行截图就可以保存整个网页了。

> 此种方案，需要安装相关环境信息：
> - Python（2或者3都行、selenium库）；
> - Chrome浏览器（以及配套的chromedriver）。

配置好相关环境变量信息（或者代码中指定相关路径）即可。

抓取到的图片效果如下：

![](/assets/post_pics/2019-07-15-html%20page%20to%20jpg.md/3.png)

## 性能考量

上面提及的两种方案，本质上都属于爬虫的一种，而且需要根据远端请求到的内容进行渲染成具体页面，再将页面转换为图片写入磁盘。

受网速、webkit渲染CPU占用、页面内容大小、IO读写等多方因素影响，其**单线程页面图片抓取的速度并不高**（在笔记本上DEMO测试的时候，百度等小页面1s以内完成，门户财经相关新闻网站页面很大，加载完成并截图保存耗时7-8s，如果部署在服务器上的性能理论上会好一些）。

如果对处理性能有较高要求，可以考虑多线程并发提升性能，但是**整体单机处理性能并不会太高**，数据量特别巨大的时候可以考虑集群部署增加处理节点。

## 附录，软件包获取

> 1. **chromedriver**: [http://chromedriver.storage.googleapis.com/index.html](http://chromedriver.storage.googleapis.com/index.html)
> 
> 2. **PhantomJS**: [https://phantomjs.org/download.html](https://phantomjs.org/download.html)
> 
> 3. **Python3**: [https://www.python.org/getit/](https://www.python.org/getit/)



---

欢迎关注我的公众号“**架构笔录**”，原创技术文章第一时间推送，也可互动一起探讨交流技术。

<center>

   ![](https://raw.githubusercontent.com/veezean/pic_assets/master/assets/comm_pics/contact/gongzhonghao.png)

</center>

