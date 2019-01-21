#架构概览(Architecture overview)#

本文档介绍了Scrapy架构及其组件之间的交互。

##概述(Overview)##

接下来的图表展现了Scrapy的架构，包括组件及在系统中发生的数据流的概览(红色箭头所示)。 下面对每个组件都做了简单介绍，并给出了详细内容的链接。数据流如下所描述。

![](https://docs.scrapy.org/en/latest/_images/scrapy_architecture_02.png)

数据流在Scrapy中通过执行引擎控制，像这样：

  1. Engine从Spider获取初始化的Request用于爬取。
  2. Engine调度Scheduler中的Request和获取下一个要爬取的Request。
  3. Scheduler返回下一个Request给Engine。
  4. Engine发送Request给Downloader，Request穿过Downloader Middlerwares(见`process_request()`)再给Downloader。
  5. 一旦Downloader下载完毕网页，就会生成一个Response(这个网页的)，穿过Downloader Middleware(见`process_response()`)发送给Engine。
  6. Engine得到从Downloader发来的Response，然后Response穿过Spider Middleware(见`process_spider_input()`)发送给Spider。
  7. Spider处理Response，返回已爬取到的items和新的Request(接下来爬爬取的)，并穿过Spider Middleware(见`process_spider_output()`),发给Engine。
  8. Engine发送已处理的item给Item Pipeline，然后发送已处理的Requests给Schduler，并尽可能请求下一个要爬取的Request。
  9. 重复该过程(从第一步开始)直到Scheduler没有Request为止。


##组件(Components)##

###引擎(Scrapy Engine)###

引擎负责控制数据流在系统中所有组件中流动，并在相应动作发生时触发事件。 详细内容查看上面的数据流(Data Flow)部分。

###调度器(Scheduler)###

调度器从引擎接受request并将他们入队，以便之后引擎请求他们时提供给引擎。

###下载器(Downloader)###

下载器负责获取页面数据并提供给引擎，而后提供给spider。

###Spiders###

Spider是Scrapy用户编写用于分析response并提取item(即获取到的item)或额外跟进的URL的类

###项目管道(Item Pipeline)###

Item Pipeline负责处理被spider提取出来的item。典型的处理有清理、 验证及持久化(例如存取到数据库中)。


###下载中间件(Downloader middlewares)###

下载器中间件是在引擎及下载器之间的特定钩子(specific hook)，处理Engine发送给Donwloader的Request，和处理Downloader传递给引擎的response。

如果你想做下面的某一件事，使用Downloader Middleware：

  - 在请求发送给Downloader之间处理这个请求(即Scrapy发送请求到网站之前)；
  - 获取到的响应在发送给spider前处理；
  - 发送一个新的请求，而不是把得到的响应传递给spider；
  - 在没有抓取网页的情况下传递响应到spider；
  - 默默丢弃一些请求。

###蜘蛛中间价(Spider middlewares)###

Spider中间件是在引擎及Spider之间的特定钩子(specific hook)，处理spider的输入(response)和输出(items及requests)。

如果你需要做以下的事情，使用蜘蛛中间件：

  - 后期处理spider的回调函数的输出 - 增/删/改 请求或者items；
  - 后期处理 start_requests；
  - 处理spider异常；
  - 根据响应文本为请求调用errback而不是callback。

##事件驱动的网络(Event-driven networking)##

Scrapy是通过Twisted基础上编写的，Twisted是一个受欢迎的事件驱动网络python框架。因此，它使用非阻塞(也就叫异步)代码实现并发。

