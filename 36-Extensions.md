#Extensions#

扩展框架提供一个机制，使得你能将自定义功能绑定到Scrapy。

扩展只是正常的类，它们在Scrapy启动时被实例化、初始化。

##扩展设置(Extension settings)##

扩展使用 Scrapy settings 管理它们的设置，这跟其他Scrapy代码一样。

通常扩展需要给它们的设置加上前缀，以避免跟已有(或将来)的扩展冲突。 比如，一个扩展处理 Google Sitemaps， 则可以使用类似 `GOOGLESITEMAP_ENABLED`、`GOOGLESITEMAP_DEPTH` 等设置。

##加载和激活扩展(Loading & activating extensions)##

扩展在扩展类被实例化时加载和激活。 因此，所有扩展的实例化代码必须在类的构造函数(<font color=red>`__init__`</font>)中执行。

要使得扩展可用，需要把它添加到Scrapy的 `EXTENSIONS` 配置中。 在 `EXTENSIONS` 中，每个扩展都使用一个字符串表示，即扩展类的全Python路径。 比如:

	EXTENSIONS = {
	    'scrapy.extensions.corestats.CoreStats': 500,
	    'scrapy.extensions.telnet.TelnetConsole': 500,
	}

如你所见，`EXTENSIONS` 配置是一个dict，key是扩展类的路径，value是顺序, 它定义扩展加载的顺序。`EXTENSIONS` 与 定义在Scrapy中的 `EXTENSIONS_BASE` 合并，然后获得最终可用扩展的顺序。

由于扩展通常不相互依赖，因此在大多数情况下，它们的加载顺序无关紧要。这就是EXTENSIONS_BASE设置定义所有扩展具有相同order(<font color=red>`0`</font>)的原因。但是，如果您需要添加依赖于已加载的其他扩展的扩展，则可以利用此功能。

##可用，启用和禁用的扩展(Available, enabled and disabled extensions)##

并不是所有可用的扩展都会被开启。一些扩展经常依赖一些特别的配置。 比如，HTTP Cache扩展是可用的但默认是禁用的，除非 `HTTPCACHE_ENABLED` 配置项设置了。

##禁用扩展(Disabling an extension)##

为了禁用一个默认开启的扩展(比如，包含在 EXTENSIONS_BASE 中的扩展)， 需要将其顺序(order)设置为 <font color=red>`None`</font> 。比如:

	EXTENSIONS = {
	    'scrapy.extensions.corestats.CoreStats': None,
	}

##编写你的扩展(Writing your own extension)##

每个扩展都是一个Python类。Scrapy扩展(包括middlewares和pipelines)的主要入口是 from_crawler 类方法， 它接收一个 Crawler 类的实例。你可以通过这个对象访问settings，signals，stats，控制爬虫的行为。

通常来说，扩展关联到 signals 并执行它们触发的任务。

最后，如果 <font color=red>`from_crawler`</font> 方法抛出  `NotConfigured` 异常， 扩展会被禁用。否则，扩展会被开启。

##扩展例子(Sample extension)##

  - spider被打开
  - spider被关闭
  - 爬取了特定数量的条目(items)

该扩展通过 <font color=red>`MYEXT_ENABLED`</font> 配置项开启， items的数量通过 <font color=red>`MYEXT_ITEMCOUNT`</font> 配置项设置。

以下是扩展的代码:

	from scrapy import signals
	from scrapy.exceptions import NotConfigured
	
	class SpiderOpenCloseLogging(object):
	
	    def __init__(self, item_count):
	        self.item_count = item_count
	
	        self.items_scraped = 0
	
	    @classmethod
	    def from_crawler(cls, crawler):
	        # first check if the extension should be enabled and raise
	
	        # NotConfigured otherwise
	
	        if not crawler.settings.getbool('MYEXT_ENABLED'):
	
	            raise NotConfigured
	
	        # get the number of items from settings
	
	        item_count = crawler.settings.getint('MYEXT_ITEMCOUNT', 1000)
	
	        # instantiate the extension object
	
	        ext = cls(item_count)
	
	        # connect the extension object to signals
	
	        crawler.signals.connect(ext.spider_opened, signal=signals.spider_opened)
	
	        crawler.signals.connect(ext.spider_closed, signal=signals.spider_closed)
	
	        crawler.signals.connect(ext.item_scraped, signal=signals.item_scraped)
	
	        # return the extension object
	
	        return ext
	
	    def spider_opened(self, spider):
	        spider.log("opened spider %s" % spider.name)
	
	    def spider_closed(self, spider):
	        spider.log("closed spider %s" % spider.name)
	
	    def item_scraped(self, item, spider):
	        self.items_scraped += 1
	        if self.items_scraped % self.item_count == 0:
	            spider.log("scraped %d items" % self.items_scraped)

##内置扩展介绍(Built-in extensions reference)##

###通用扩展(General purpose extensions)###

#####记录统计扩展(Log Stats extension)#####

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.extensions.logstats.LogStats
</td></tr></table>
记录基本的统计信息，比如爬取的页面和条目(items)。

#####核心统计扩展(Core Stats extension)#####
<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.extensions.corestats.CoreStats
</td></tr></table>
如果统计收集器(stats collection)启用了，该扩展开启核心统计收集(参考 数据收集(Stats Collection))。

#####Telnet console 扩展(Telnet console extension)#####


<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.extensions.telnet.TelnetConsole
</td></tr></table>

提供一个telnet控制台，用于进入当前执行的Scrapy进程的Python解析器， 这对代码调试非常有帮助。

telnet控制台通过 `TELNETCONSOLE_ENABLED` 配置项开启， 服务器会监听 `TELNETCONSOLE_PORT` 指定的端口。


#####内存使用扩展(Memory usage extension)#####


<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.extensions.memusage.MemoryUsage
</td></tr></table>

<font color=orange>NOTE：</br>wondows中此扩展不工作</font>


监控Scrapy进程内存使用量，并且：

  1. 如果使用内存量超过某个指定值，发送提醒邮件
  2. 如果超过某个指定值，关闭spider

当内存用量达到 `MEMUSAGE_WARNING_MB` 指定的值，发送提醒邮件。 当内存用量达到 `MEMUSAGE_LIMIT_MB` 指定的值，发送提醒邮件，同时关闭spider， Scrapy进程退出。

该扩展通过 `MEMUSAGE_ENABLED` 配置项开启，可以使用以下选项：

  - `MEMUSAGE_LIMIT_MB`
  - `MEMUSAGE_WARNING_MB`
  - `MEMUSAGE_NOTIFY_MAIL`
  - `MEMUSAGE_CHECK_INTERVAL_SECONDS`



#####内存调试扩展(Memory debugger extension)#####

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.extensions.memdebug.MemoryDebugger
</td></tr></table>

该扩展用于调试内存使用量，它收集以下信息：

  - 没有被Python垃圾回收器收集的对象
  - 应该被销毁却仍然存活的对象。更多信息请参考 使用 trackref 调试内存泄露

开启该扩展，需打开 `MEMDEBUG_ENABLED` 配置项。 信息将会存储在统计信息(stats)中。

#####关闭spider扩展(Close spider extension)#####


<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.extensions.closespider.CloseSpider
</td></tr></table>

当某些状况发生，spider会自动关闭。每种情况使用指定的关闭原因。

关闭spider的情况可以通过下面的设置项配置：

  - `CLOSESPIDER_TIMEOUT`
  - `CLOSESPIDER_ITEMCOUNT`
  - `CLOSESPIDER_PAGECOUNT`
  - `CLOSESPIDER_ERRORCOUNT`

**CLOSESPIDER_TIMEOUT**

默认：<font color=red>`0`</font>

一个整数值，单位为秒。如果一个spider在指定的秒数后仍在运行， 它将以 <font color=red>`closespider_timeout`</font> 的原因被自动关闭。 如果值设置为0（或者没有设置），spiders不会因为超时而关闭。

**CLOSESPIDER_ITEMCOUNT**

默认：<font color=red>`0`</font>

一个整数值，指定条目的个数。如果spider爬取条目数超过了指定的数， 并且这些条目通过item pipeline传递，spider将会以 <font color=red>`closespider_itemcount`</font> 的原因被自动关闭。当前仍在下载队列(相当于 `CONCURRENT_REQUESTS` 的请求) 仍然会被处理。如果值设置为0（或者没有设置），spiders不会因为item数量而关闭。

**CLOSESPIDER_PAGECOUNT**

默认：<font color=red>`0`</font>

一个整数值，指定最大的抓取响应(reponses)数。 如果spider抓取数超过指定的值，则会以 <font color=red>`closespider_pagecount`</font> 的原因自动关闭。 如果设置为0（或者未设置），spiders不会因为抓取的响应数而关闭。


**CLOSESPIDER_ERRORCOUNT**

默认：<font color=red>`0`</font>

一个整数值，指定spider可以接受的最大错误数。 如果spider生成多于该数目的错误，它将以 <font color=red>`closespider_errorcount`</font> 的原因关闭。 如果设置为0（或者未设置），spiders不会因为发生错误过多而关闭。


#####(StatsMailer extension)#####


<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.extensions.statsmailer.StatsMailer
</td></tr></table>

这个简单的扩展可用来在一个域名爬取完毕时发送提醒邮件， 包含Scrapy收集的统计信息。 邮件会发送个通过 `STATSMAILER_RCPTS` 指定的所有接收人。

##Debugging extensions##

#####Stack trace dump extension#####

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.extensions.debug.StackTraceDump
</td></tr></table>

当收到 SIGQUIT 或 SIGUSR2 信号，spider进程的信息将会被存储下来。 存储的信息包括：

  1. engine状态(使用 <font color=red>`scrapy.utils.engin.get_engine_status()`</font>)
  2. 所有存活的引用(live references)(参考 使用 trackref 调试内存泄露)
  3. 所有线程的堆栈信息

当堆栈信息和engine状态存储后，Scrapy进程继续正常运行。

该扩展只在POSIX兼容的平台上可运行（比如不能在Windows运行）， 因为 SIGQUIT 和 SIGUSR2 信号在Windows上不可用。

至少有两种方式可以向Scrapy发送 SIGQUIT 信号:

  1. 在Scrapy进程运行时通过按Ctrl-(仅Linux可行?)
  2. 运行该命令(<font color=red>`<pid>`</font>) 是Scrapy运行的进程):

		kill -QUIT <pid>

#####调试扩展Debugger extension#####

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.extensions.debug.Debugger
</td></tr></table>

当收到 SIGUSR2 信号，将会在Scrapy进程中调用 Python debugger 。 debugger退出后，Scrapy进程继续正常运行。

更多信息参考 Debugging in Python 。

该扩展只在POSIX兼容平台上工作(比如不能再Windows上运行)。