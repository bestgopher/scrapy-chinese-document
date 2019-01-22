#Signals#

Scrapy使用信号来通知事情发生。您可以在您的Scrapy项目中捕捉一些信号(使用 extension)来完成额外的工作或添加额外的功能，扩展Scrapy。

虽然信号提供了一些参数，不过处理函数不用接收所有的参数 - 信号分发机制(singal dispatching mechanism)仅仅提供处理器(handler)接受的参数。

您可以通过 信号(Signals) API 来连接(或发送您自己的)信号。

这是一个简单的示例，展示了如何捕获信号并执行某些操作：

from scrapy import signals
from scrapy import Spider


class DmozSpider(Spider):
    name = "dmoz"
    allowed_domains = ["dmoz.org"]
    start_urls = [
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/",
    ]


    @classmethod
    def from_crawler(cls, crawler, *args, **kwargs):
        spider = super(DmozSpider, cls).from_crawler(crawler, *args, **kwargs)
        crawler.signals.connect(spider.spider_closed, signal=signals.spider_closed)
        return spider


    def spider_closed(self, spider):
        spider.logger.info('Spider closed: %s', spider.name)


    def parse(self, response):
        pass

##延迟的信号处理器(Deferred signal handlers)##

有些信号支持从处理器返回 Twisted deferreds ，参考下边的 内置信号参考手册(Built-in signals reference) 来了解哪些支持。

##内置信号参考手册(Built-in signals reference)##
以下给出Scrapy内置信号的列表及其意义。

####engine_started####

<table><tr><td>
scrapy.signals.engine_started()
</td></tr></table>

当Scrapy引擎启动爬取时发送该信号。

该信号支持返回deferreds。

<font color=orange>
NOTE:</br>
该信号可能会在信号 spider_opened 之后被发送，取决于spider的启动方式。 所以不要 依赖 该信号会比 spider-opened 更早被发送。
</font>

####engine_stopped####
<table><tr><td>
scrapy.signals.engine_stopped()
</td></tr></table>

当Scrapy引擎停止时发送该信号(例如，爬取结束)。

该信号支持返回deferreds。

####item_scraped####
<table><tr><td>
scrapy.signals.item_scraped(item, response, spider)
</td></tr></table>

当item被爬取，并通过所有 Item Pipeline 后(没有被丢弃(dropped)，发送该信号。

该信号支持返回deferreds。

&nbsp;&nbsp;&nbsp;**参数**：

  - **item** (`Item` 对象) – 爬取到的item
  - **spider** (`Spider` 对象) – 爬取item的spider
  - **response** (`Response` 对象) – 提取item的response
####item_dropped####
<table><tr><td>
scrapy.signals.engine_stopped()
</td></tr></table>

当item通过 Item Pipeline ，有些pipeline抛出 DropItem 异常，丢弃item时，该信号被发送。

该信号支持返回deferreds。

&nbsp;&nbsp;&nbsp;**参数**：

  - **item** (`Item` 对象) – Item Pipeline 丢弃的item
  - **spider** (`Spider` 对象) – 爬取item的spider
  - **response**(`response` 对象) - 获得item的响应
  - **exception** (`DropItem` 异常) – 导致item被丢弃的异常(必须是 DropItem 的子类)



####spider_closed####
<table><tr><td>
scrapy.signals.spider_closed(spider, reason)
</td></tr></table>

当某个spider被关闭时，该信号被发送。该信号可以用来释放每个spider在 spider_opened 时占用的资源。

该信号支持返回deferreds。


&nbsp;&nbsp;&nbsp;**参数**：

  - **spider** (`Spider` 对象) – 关闭的spider
  - **reason** (str) – 描述spider被关闭的原因的字符串。如果spider是由于完成爬取而被关闭，则其为 <font color-red>`'finished'`</font> 。否则，如果spider是被引擎的 <font color=red>`close_spider`</font> 方法所关闭，则其为调用该方法时传入的<font color=red>`reason`</font> 参数(默认为 <font color=red>`'cancelled'`</font>)。如果引擎被关闭(例如， 输入Ctrl-C)，则其为 <font color=red>`'shutdown'`</font> 。

####spider_opened####
<table><tr><td>
scrapy.signals.spider_opened(spider)
</td></tr></table>

当spider开始爬取时发送该信号。该信号一般用来分配spider的资源，不过其也能做任何事。

该信号支持返回deferreds。

&nbsp;&nbsp;&nbsp;**参数**：

  - **spider** (`Spider` 对象) – 开启的spider

####spider_idle####
<table><tr><td>
scrapy.signals.spider_idle(spider)
</td></tr></table>

当spider进入空闲(idle)状态时该信号被发送。空闲意味着:

  - requests正在等待被下载
  - requests被调度
  - items正在item pipeline中被处理

当该信号的所有处理器(handler)被调用后，如果spider仍然保持空闲状态， 引擎将会关闭该spider。当spider被关闭后， `spider_closed` 信号将被发送。

你可以抛出一个 `DontCloseSpider` 异常来阻止spider被关闭

该信号 `不支持` 返回deferreds。

&nbsp;&nbsp;&nbsp;**参数**：

  - **spider**(`Spider` 对象) – 空闲的spider

####spider_error####
<table><tr><td>
scrapy.signals.spider_error(failure, response, spider)
</td></tr></table>

当spider的回调函数产生错误时(例如，抛出异常)，该信号被发送。

该信号 `不支持` 返回deferreds。


&nbsp;&nbsp;&nbsp;**参数**：

  - **failure** (`Failure` 对象) – 以Twisted Failure 对象抛出的异常
  - **response** (`Response` 对象) – 当异常被抛出时被处理的response
  - **spider** (`Spider` 对象) – 抛出异常的spider

####request_scheduled####
<table><tr><td>
scrapy.signals.request_scheduled(request, spider)
</td></tr></table>

当引擎调度一个 Request 对象用于下载时，该信号被发送。

该信号 `不支持` 返回deferreds。

&nbsp;&nbsp;&nbsp;**参数**：

  - **request** (`Request` 对象) – 到达调度器的request
  - **spider** (`Spider` 对象) – 产生该request的spider


####request_dropped####
<table><tr><td>
scrapy.signals.request_dropped(request, spider)
</td></tr></table>

当一个稍后会被引擎调度下载的 `Request` 发送至调度器， 被调度器拒绝。

该信号 `不支持` 返回deferreds。

&nbsp;&nbsp;&nbsp;**参数**：

  - **request** (`Request` 对象) – 到达调度器的request
  - **spider** (`Spider` 对象) – 产生该request的spider

####response_received####
<table><tr><td>
scrapy.signals.response_received(response, request, spider)
</td></tr></table>

当引擎从downloader获取到一个新的 Response 时发送该信号。

该信号 `不支持` 返回deferreds。

&nbsp;&nbsp;&nbsp;**参数**：

  - response (`Response`对象) – 接收到的response
  - request (`Request`对象) – 生成response的request
  - spider (`Spider`对象) – response所对应的spider

####response_downloaded####
<table><tr><td>
scrapy.signals.response_downloaded(response, request, spider)
</td></tr></table>

当一个 <font color=red>`HTTPResponse`</font> 被下载时，由downloader发送该信号。

该信号 `不支持` 返回deferreds。

&nbsp;&nbsp;&nbsp;**参数**：

  - response (`Response`对象) – 接收到的response
  - request (`Request`对象) – 生成response的request
  - spider (`Spider`对象) – response所对应的spider