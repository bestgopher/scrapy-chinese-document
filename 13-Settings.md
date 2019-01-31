#Settings#

Scrapy设定(settings)提供了定制Scrapy组件的方法。您可以控制包括核心(core)，插件(extension)，pipeline及spider组件。

设定为代码提供了提取以key-value映射的配置值的的全局命名空间(namespace)。 设定可以通过下面介绍的多种机制进行设置。

设定(settings)同时也是选择当前激活的Scrapy项目的方法(如果您有多个的话)。

内置设定列表请参考 内置设定参考手册 。

#指定设定(Designating the settings)#

当您使用Scrapy时，您需要声明您所使用的设定。这可以通过使用环境变量: <font color=red>`SCRAPY_SETTINGS_MODULE`</font> 来完成。

<font color=red>`SCRAPY_SETTINGS_MODULE`</font> 必须以Python路径语法编写, 如 <font color=red>`myproject.settings`</font> 。 注意，设定模块应该在 Python `import search path` 中。

#获取设定值(Populating the settings)#

设定可以通过多种方式设置，每个方式具有不同的优先级。 下面以优先级降序的方式给出方式列表:

  1. 命令行选项(Command line Options)(最高优先级)
  2. 每个spider的设定
  3. 项目设定模块(Project settings module)
  4. 命令默认设定模块(Default settings per-command)
  5. 全局默认设定(Default global settings) (最低优先级)

这些设定(settings)由scrapy内部很好的进行了处理，不过您仍可以使用API调用来手动处理。 详情请参考 设置`(Settings API）`。

这些机制将在下面详细介绍。

##1. 命令行选项(Command line options)##

命令行传入的参数具有最高的优先级。 您可以使用command line 选项 -<font color=red>`s`</font> (或 <font color=red>`--set`</font>) 来覆盖一个(或更多)选项。

样例:

	scrapy crawl myspider -s LOG_FILE=scrapy.log

##2. 为每个spider设置(Settings per-spider)##

Spider可以定义自己的设置，这些设置的优先级最高，会覆盖项目的设置。它们可以通过 `custom_settings` 属性定义：

	class MySpider(scrapy.Spider):
	    name = 'myspider'
	
	    custom_settings = {
	        'SOME_SETTING': 'some value',
	    }

##3. 项目设定模块(Project settings module)##

项目设定模块是您Scrapy项目的标准配置文件。 其是获取大多数设定的方法。对于一个标准的Scrapy项目，这就意味着你添加或者修改这些设置在为你项目创建的 <font color=red>`settings.py`</font> 文件中。

##4. 命令默认设定(Default settings per-command)##

每个 `Scrapy tool` 命令拥有其默认设定，并覆盖了全局默认的设定。 这些设定在命令的类的 <font color=red>`default_settings`</font> 属性中指定。

##5. 默认全局设置(Default global settings)##

全局默认设定存储在  <font color=red>`scrapy.settings.default_setting`s</font> 模块， 并在 内置设定参考手册 部分有所记录。

#如何访问设定(How to access settings)#

在spider中，设置key通过 <font color=red>`self.settings`</font> 获得。

	class MySpider(scrapy.Spider):
	    name = 'myspider'
	    start_urls = ['http://example.com']
	
	    def parse(self, response):
	        print("Existing settings: %s" % self.settings.attributes.keys())

<font color=orange>
NOTE:</br>
<font color=red>`settings`</font> 属性是在spider实例化之后在基础Spider类中设置的。如果你想在实例化之前使用设置(例如在你spider的`__init__()` 方法中)，你需要重写 `from_crawler()` 方法
</font>

设置可以通过Crawler的 `scrapy.crawler.Crawler.settings` 属性获得，Crawler会传递给扩展(extensions)，中间件(middlewares)，项目管道(item pipeline)的 `from_crawler` 方法：
	
	class MyExtension(object):
	    def __init__(self, log_is_enabled=False):
	        if log_is_enabled:
	            print("log is enabled!")
	
	    @classmethod
	    def from_crawler(cls, crawler):
	        settings = crawler.settings
	        return cls(settings.getbool('LOG_ENABLED'))

settings对象可以像字典一样使用(例如: <font color=red>`settings['LOG_ENABLED']`</font>)，但是通常优先使用 `Settings` API提供的方法来按格式获取你需要的setting，从而避免类型错误(type errors)。

#设定名字的命名规则(Rationale for setting names)#

设定的名字以要配置的组件作为前缀。 例如，一个robots.txt插件的合适设定应该为 <font color=red>`ROBOTSTXT_ENABLED`</font>, <font color=red>`ROBOTSTXT_OBEY`</font>, <font color=red>`ROBOTSTXT_CACHEDIR`</font> 等等。

这里以字母序给出了所有可用的Scrapy设定及其默认值和应用范围。

如果给出可用范围，并绑定了特定的组件，则说明了该设定使用的地方。 这种情况下将给出该组件的模块，通常来说是插件、中间件或pipeline。 同时也意味着为了使设定生效，该组件必须被启用。


##AWS\_ACCESS\_KEY\_ID##

默认: <font color=red>`None`</font>

连接 Amazon Web services 的AWS access key。 S3 feed storage backend 中使用.

##AWS_SECRET\_ACCESS\_KEY##

默认: <font color=red>`None`</font>

连接 Amazon Web services 的AWS secret key。 S3 feed storage backend 中使用。

##BOT_NAME##

默认: <font color=red>`scrapybot`</font>

Scrapy项目实现的bot的名字(也为项目名称)。 这将用来构造默认 User-Agent，同时也用来log。

当您使用 `startproject` 命令创建项目时其也被自动赋值。

##CONCURRENT_ITEMS##

默认: <font color=red>`100`</font>

Item Processor(即 Item Pipeline) 同时并行处理(每个response的)item的最大值。

##CONCURRENT_REQUESTS##

默认: <font color=red>`16`</font>

Scrapy downloader 并发请求(concurrent requests)的最大值。

##CONCURRENT\_REQUESTS\_PER\_DOMAIN##


默认: <font color=red>`8`</font>

对单个网站进行并发请求的最大值。

另请查阅：AutoThrottle extension和它的`AUTOTHROTTLE_TARGET_CONCURRENCY`选项。

##CONCURRENT\_REQUESTS\_PER\_IP##

默认: <font color=red>`0`</font>

对单个IP进行并发请求的最大值。如果非0，则忽略 `CONCURRENT_REQUESTS_PER_DOMAIN` 设定， 使用该设定。 也就是说，并发限制将针对IP，而不是网站。

该设定也影响 `DOWNLOAD_DELAY`和AutoThrottle extension: 如果 `CONCURRENT_REQUESTS_PER_IP` 非0，下载延迟应用在IP而不是网站上。

##DEFAULT\_ITEM\_CLASS##

默认: <font color=red>`'scrapy.item.Item'`</font>

the Scrapy shell 中实例化item使用的默认类。

##DEFAULT\_REQUEST\_HEADERS##
默认：
	{
	    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
	    'Accept-Language': 'en',
	}

Scrapy HTTP Request使用的默认header。由 `DefaultHeadersMiddleware` 产生。

##DEPTH_LIMIT##

默认: <font color=red>`0`</font>

应用处：<font color=red>`scrapy.spidermiddlewares.depth.DepthMiddleware`</font>

爬取网站最大允许的深度(depth)值。如果为0，则没有限制。

##DEPTH_PRIORITY##

默认: <font color=red>`0`</font>

应用处：<font color=red>`scrapy.spidermiddlewares.depth.DepthMiddleware`</font>

整数值。用于根据深度调整request优先级：

  - 如果为0(默认)，则不根据深度进行优先级调整。
  - 正值将会减少优先级，即更深的请求越晚处理；通常在广度优先(BFO)时使用
  - 负值将会增加优先级，即更深的请求越早处理(深度优先)

<font color=orange>
NOTE:</br>
这个设置优先级相较于其他优先级设置 `REDIRECT_PRIORITY_ADJUST` 和 `RETRY_PRIORITY_ADJUST`，使用的相反的方式。
</font>

##DEPTH_STATS##

默认: <font color=red>True</font>

应用处：<font color=red>`scrapy.spidermiddlewares.depth.DepthMiddleware`</font>

是否收集最大深度数据。

##DEPTH\_STATS\_VERBOSE##

默认: <font color=red>False</font>

应用处：<font color=red>`scrapy.spidermiddlewares.depth.DepthMiddleware`</font>

是否收集详细的深度数据。如果启用，每个深度的请求数将会被收集在数据中。

##DNSCACHE_ENABLED##

默认: <font color=red>True</font>

是否启用DNS内存缓存(DNS in-memory cache)。

##DNSCACHE_SIZE##

默认: <font color=red>10000</font>

DNS内存缓存大小。

##DNS_TIMEOUT##

默认: <font color=red>60</font>

处理DNS查询时，超时的秒数。支持浮点数。

##DOWNLOADER##

默认: <font color=red>`'scrapy.core.downloader.Downloader'`</font>

用于crawl的下载器(downloader).

##DOWNLOADER_HTTPCLIENTFACTORY##

默认: <font color=red>`'scrapy.core.downloader.webclient.ScrapyHTTPClientFactory'`</font>

定义一个Twisted <font color=red>protocol.ClientFactory</font> 类，用于HTTP/1.0连接(为 <font color=red>`HTTP10DownloadeHandler`</font>)

<font color=orange>
NOTE：</br>
今天HTTP/1.0已经很少使用了，因此你可以安全地忽略这个选项，除非你使用的Twisted<11.1，或者你真的想使用HTTP/1.0，为<font color=red>`http(s)`</font>相应的机制重写了 `DOWNLOAD_HANDLERS_BASE`
</font>，即 <font color=red>`'scrapy.core.downloader.handlers.http.HTTP10DownloadHandler'`</font>

##DOWNLOADER_CLIENTCONTEXTFACTORY##

默认: <font color=red>`'scrapy.core.downloader.contextfactory.ScrapyClientContextFactory'`</font>

表示要使用的ContextFactory类的路径。

这里，"ContextFactory"是SSL/TLS上下文中的Twisted术语，定义TLS/SSL协议版本，是否证书验证，或者客户端的身份验证(和各种各样其他事情)

<font color=orange>
NOTE：
Scrapy默认的上下文工厂**不会执行远程服务器的证书验证**。这样通常有益于网页爬取。

如果你需要启用远程服务器的证书验证，Scrapy也有其他上下文工厂供你设置，<font color=red>`'scrapy.core.downloader.contextfactory.BrowserLikeContextFactory'`</font>，使用平台的证书来验证远程端点。**这个只在Twisted>=14.0版本可用**
</font>

如果你使用自定义的ContextFactory，确保在init中能够接受<font color=red>`method`</font> 参数(这是 <font color=red>`OpenSSL.SSL`</font>方法的映射 `DOWNLOADER_CLIENT_TLS_METHOD`)

##DOWNLOADER\_CLIENT\_TLS\_METHOD##

默认：<font color=red>`TLS`</font>

使用这个设置定义HTTP/1.1下载器默认使用的TLS/SSL方法。

这个设置必须为下面字符串之一：

  - <font color=red>`TLS`</font>：映射到 OpenSSL 的 <font color=red>`TLS_method()`</font> (又名 <font color=red>`SSLv23_method()`</font>)，它允许协议协商，从平台支持的最高版本开始；**默认值，推荐使用这个**。
  - <font color=red>`TLSv1.0`</font>：这个值强制HTTPS通过使用TLS1.0版本连接；如果你想Scrapy<1.1版本的行为就可以设置为这个。
  - <font color=red>`TLSv1.1`</font>：强制TLS1.1版本
  - <font color=red>`TLSv1.2`</font>：强制TLS1.2版本
  - <font color=red>`SSLv3`</font>：强制SSL版本3(**不推荐**)

<font color=orange>
NOTE：</br>
我们建议您使用PyOpenSSL> = 0.13和Twisted> = 0.13或更高（如果可以，Twisted> = 14.0）。
</font>

##DOWNLOADER_MIDDLEWARES##

默认: <font color=red>`{}`</font>

保存项目中启用的下载中间件及其顺序的字典。

##DOWNLOADER\_MIDDLEWARES\_BASE##

默认：

	{
	    'scrapy.downloadermiddlewares.robotstxt.RobotsTxtMiddleware': 100,
	    'scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware': 300,
	    'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware': 350,
	    'scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware': 400,
	    'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': 500,
	    'scrapy.downloadermiddlewares.retry.RetryMiddleware': 550,
	    'scrapy.downloadermiddlewares.ajaxcrawl.AjaxCrawlMiddleware': 560,
	    'scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware': 580,
	    'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 590,
	    'scrapy.downloadermiddlewares.redirect.RedirectMiddleware': 600,
	    'scrapy.downloadermiddlewares.cookies.CookiesMiddleware': 700,
	    'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware': 750,
	    'scrapy.downloadermiddlewares.stats.DownloaderStats': 850,
	    'scrapy.downloadermiddlewares.httpcache.HttpCacheMiddleware': 900,
	}

包含Scrapy默认启用的下载中间件的字典。低序更靠近引擎，高序更靠近下载器。永远不要在项目中修改该设定，而是修改 `DOWNLOADER_MIDDLEWARES`。

##DOWNLOADER_STATS##

默认：<font color=red>`True`</font>

是否收集下载器数据。

##DOWNLOAD_DELAY##

默认：<font color=red>`0`</font>

下载器在下载同一个网站下一个页面前需要等待的时间。该选项可以用来限制爬取速度， 减轻服务器压力。同时也支持小数：

	DOWNLOAD_DELAY = 0.25    # 250 ms of delay

该设定影响(默认启用的) `RANDOMIZE_DOWNLOAD_DELAY` 设定。 默认情况下，Scrapy在两个请求间不等待一个固定的值， 而是使用0.5到1.5之间的一个**随机值乘以 DOWNLOAD_DELAY** 的结果作为等待间隔。

当 `CONCURRENT_REQUESTS_PER_IP` 非0时，延迟针对的是每个ip而不是网站。

另外您可以通过spider的 <font color=red>`download_delay`</font> 属性为每个spider设置该设定。

##DOWNLOAD_HANDLERS##

默认：<font color=red>`{}`</font>

保存项目中启用的下载处理器(request downloader handler)的字典。 例子请查看 `DOWNLOAD_HANDLERS_BASE` 。

##DOWNLOAD\_HANDLERS\_BASE##

默认：
	
	{
	    'file': 'scrapy.core.downloader.handlers.file.FileDownloadHandler',
	    'http': 'scrapy.core.downloader.handlers.http.HTTPDownloadHandler',
	    'https': 'scrapy.core.downloader.handlers.http.HTTPDownloadHandler',
	    's3': 'scrapy.core.downloader.handlers.s3.S3DownloadHandler',
	    'ftp': 'scrapy.core.downloader.handlers.ftp.FTPDownloadHandler',
	}

保存项目中默认启用的下载处理器(request downloader handler)的字典。 永远不要在项目中修改该设定，而是修改 `DOWNLOADER_HANDLERS` 。

如果需要关闭上面的下载处理器，您必须在项目中的 DOWNLOAD_HANDLERS 设定中设置该处理器，并为其赋值为 None 。 例如，关闭(而不是替换)FTP处理器，在 <font color=red>`settings.py`</font>中:

	DOWNLOAD_HANDLERS = {
	    'ftp': None,
	}

##DOWNLOAD_TIMEOUT##

默认：<font color=red>`100`</font>

下载器超时时间(单位: 秒)。

<font color=orange>
NOTE：</br>
这个超时时间可以使用`download_timeout`属性为每个spider设置，也可以使用`download_timeout`作为Request.meta.key为每个Request设置。
</font>

##DOWNLOAD_MAXSIZE##

默认：<font color=red>`1073741824 (1024MB)`</font>

下载器将会下载的响应最大尺寸(单位bytes)。

如果你想禁用它，设置为0

<font color=orange>
NOTE：</br>
这个大小可以通过`download_maxsize`属性为每个spider设置，也可以使用`download_maxsize`作为Request.meta.key为每个Request设置。
这需要Twisted>=11.1
</font>

##DOWNLOAD_WARNSIZE##

默认：<font color=red>`33554432 (32MB)`</font>

设置响应的大小(单位bytes)超过这个值，下载器开始警告。

如果你想禁用它，设置为0

<font color=orange>
NOTE：</br>
这个大小可以通过`download_warnsize`属性为每个spider设置，也可以使用`download_warnsize`作为Request.meta.key为每个Request设置。
这需要Twisted>=11.1

</font>

##DOWNLOAD\_FAIL\_ON\_DATALOSS##

默认：<font color=red>`True`</font>

不完整的response是否为失败，也就是说，声明的<font color=red>`Content-Length`</font>与服务器发送过来的文本长度不匹配，或者response块没有正确完成。如果为<font color=red>True</font>，这些response将会抛出<font color=red>`ResponseFailed([_DataLoss])`</font>错误。如果为<font color=red>False</font>，这些response被正常传递，但是<font color=red>`dataloss`</font>将会加到response中，即<font color=red>`'dataloss' in response.flags`</font> is <font color=red>True</font> 。

你可以随意的为每个请求设置，通过 设置Request.meta的key为`download_fail_on_dataloss`，值为`False`。

<font color=orange>
NOTE：</br>
不完整响应，或者数据丢失错误，可能发生在以下几种情况，服务器错误配置、网络错误、数据损坏。考虑到不完整的repsonse可能包含部分或者不完整的文本，所以由用户决定处理不完整response是否有意义。如果 `RETRY_ENABLED` 是 <font color=red>`True`</font> ，而且这个选项也设置为 <font color=red>`True`</font> ，<font color=red>`ResponseFailed([_DataLoss])`</font> 错误通常将会再次请求。
</font>

##DUPEFILTER_CLASS##

默认：<font color=red>`'scrapy.dupefilters.RFPDupeFilter'`</font>

用于检测过滤重复请求的类。

默认的 (<font color=red>`RFPDupeFilter`</font>) 过滤器基于 <font color=red>`scrapy.utils.request.request_fingerprint `</font>函数生成的请求<font color=red>`fingerprint(指纹)`</font>。 如果您需要修改检测的方式，您可以继承 <font color=red>`RFPDupeFilter`</font> 并覆盖其 <font color=red>`request_fingerprint`</font> 方法。 该方法接收 Request 对象并返回其fingerprint(一个字符串)。

你可以通过设置 `DUPEFILTER_CLASS` 为 <font color=red>`scrapy.dupefilters.BaseDupeFilter`</font> 来禁用重复过滤。你需要小心这样设置，因为你可能进入爬取循环中。更好的方法是设置 <font color=red>`dont_filter`</font> 为 <font color=red>`True`</font> 来为特定的 `Request` 设置不去重。

##DUPEFILTER_DEBUG##

默认：<font color=red>`False`</font>

默认情况下， <font color=red>`RFPDupeFilter`</font> 只记录第一次重复的请求。 设置 `DUPEFILTER_DEBUG` 为 <font color=red>`True`</font> 将会使其记录所有重复的requests。

##EDITOR##

默认：<font color=red>`vi`</font>(Unix 系统)，<font color=red>`IDLE`</font>(windows)

执行 `edit` 命令编辑spider时使用的编辑器。此外，如果设置了 <font color=red>`EDITOR`</font> 环境变量，`edit` 命令将优先使用这个设置的编辑器。

##EXTENSIONS##

默认：<font color=red>`{}`</font>

保存项目中启用的插件及其顺序的字典。

##EXTENSIONS_BASE##

默认：

	{
	    'scrapy.extensions.corestats.CoreStats': 0,
	    'scrapy.extensions.telnet.TelnetConsole': 0,
	    'scrapy.extensions.memusage.MemoryUsage': 0,
	    'scrapy.extensions.memdebug.MemoryDebugger': 0,
	    'scrapy.extensions.closespider.CloseSpider': 0,
	    'scrapy.extensions.feedexport.FeedExporter': 0,
	    'scrapy.extensions.logstats.LogStats': 0,
	    'scrapy.extensions.spiderstate.SpiderState': 0,
	    'scrapy.extensions.throttle.AutoThrottle': 0,
	}

包含默认可用的插件及其顺序的字典。该设定包含所有稳定(stable)的内置插件。需要注意，有些插件需要通过设定来启用。

更多内容请参考 extensions用户手册 及 所有可用的插件 。

##FEED_TEMPDIR##

Feed Temp目录允许你自定义在使用FTP feed storage和Amazon S3下载的时候，保存的文件的临时文件夹。

##FTP\_PASSIVE\_MODE##

默认：<font color=red>`True`</font>

启动FTP传输时是否启用被动模式。

##FTP_PASSWORD##

默认：<font color=red>`"guest"`</font>

<font color=red>`Resquest.meta`</font> 中没有 <font color=red>`"ftp_password"`</font> 时，使用这个作为FTP连接的密码。

<font color=orange>
NOTE：</br>
根据RFC 1635解释，虽然通常使用密码'guest'或者某个匿名FTP的邮箱地址，一些FTP服务器明确询问用户的邮箱地址，而且将不允许使用'guest'密码登录。
</font>

##FTP_USER##

默认：<font color=red>`"anonymous"`</font>

<font color=red>`Resquest.meta`</font> 中没有 <font color=red>`"ftp_user"`</font> 时，使用这个作为FTP连接的用户名。

##ITEM_PIPELINES##

默认：<font color=red>`{}`</font>

一个包含能够使用的pipeline和他们使用顺序的字典。顺序值是任意的，但是习惯上定义 `0-1000` 之间。数字小的比数字高的先处理item。

举例：

	ITEM_PIPELINES = {
	    'mybot.pipelines.validate.ValidateMyItem': 300,
	    'mybot.pipelines.validate.StoreMyItem': 800,
	}

##ITEM\_PIPELINES\_BASE##

默认：<font color=red>`{}`</font>

保存项目中默认启用的pipeline的字典。 永远不要在项目中修改该设定，而是修改 `ITEM_PIPELINES` 。

##LOG_ENABLED##

默认：<font color=red>`True`</font>

是否启用logging。

##LOG_ENCODING##

默认：<font color=red>`'utf-8'`</font>

logging使用的编码。

##LOG_FILE##

默认：<font color=red>`None`</font>

logging输出的文件名。如果为<font color=red>`'None'`</font>，则使用标准错误输出(standard error)。

##LOG_FORMAT##

默认：<font color=red>`'%(asctime)s [%(name)s] %(levelname)s: %(message)s'`</font>

定义日志信息格式的字符串。

##LOG_DATEFORMAT##

默认：<font color=red>`'%Y-%m-%d %H:%M:%S'`</font>

定义日期/时间的字符串，也就是 `LOG_FORMAT` 中 <font color=red>`'%(asctime)s'`</font> 的占位符。

##LOG_LEVEL##

默认：<font color=red>`'DEBUG'`</font>

log的最低级别。可选的级别有: CRITICAL、 ERROR、WARNING、INFO、DEBUG。

##LOG_STDOUT##


默认：<font color=red>`False`</font>

如果为 <font color=red>`True`</font> ，进程所有的标准输出(及错误)将会被重定向到log中。例如， 执行 <font color=red>`print 'hello'</font> ，其将会在Scrapy log中显示。

##LOG\_SHORT\_NAMES##

默认：<font color=red>`False`</font>

如果 <font color=red>`True`</font>，日志将只包含根路径。如果设置为 <font color=red>`False`</font> 则显示负责日志输出的组件

##MEMDEBUG_ENABLED##

默认：<font color=red>`False`</font>

是否启用内存调试(memory debugging)。

##MEMDEBUG_NOTIFY##

默认：<font color=red>`[]`</font>

如果该设置不为空，当启用内存调试时将会发送一份内存报告到指定的地址；否则该报告将写到log中。

样例:

	MEMDEBUG_NOTIFY = ['user@example.com']

==================

##MEMUSAGE_ENABLED##

默认：<font color=red>`True`</font>

应用于：<font color=red>`scrapy.extensions.memusage`</font>

是否启用内存使用量监控扩展。此扩展程序跟踪进程使用的内存峰值（将其写入统计信息）。它还可以选择在超出内存限制时关闭Scrapy进程（请参阅参考资料`MEMUSAGE_LIMIT_MB`），并在发生时通过电子邮件通知（请参阅参考资料`MEMUSAGE_NOTIFY_MAIL`）

##MEMUSAGE\_LIMIT\_MB##

默认：<font color=red>`0`</font>

应用于：<font color=red>`scrapy.extensions.memusage`</font>

在关闭Scrapy之前所允许使用的最大内存数(单位: MB)(如果 `MEMUSAGE_ENABLED`为True)。 如果为0，将不做限制。

##MEMUSAGE\_CHECK\_INTERVAL\_SECONDS##

默认：<font color=red>`60.0`</font>

应用于：<font color=red>`scrapy.extensions.memusage`</font>

每隔一个固定的时间间隔，内存管理扩展检查当前内存使用量，在与`MEMUSAGE_LIMIT_MB` 和 `MEMUSAGE_WARNING_MB` 设置的值比较。

这个设置间隔时间的长度，单位为秒。

##MEMUSAGE\_NOTIFY\_MAIL##


默认：<font color=red>`False`</font>

应用于：<font color=red>`scrapy.extensions.memusage`</font>

达到内存限制时通知的email的列表。

样例：

	MEMUSAGE_NOTIFY_MAIL = ['user@example.com']

##MEMUSAGE\_WARNING\_MB##

默认：<font color=red>`0`</font>

应用于：<font color=red>`scrapy.extensions.memusage`</font>

在发送警告email前所允许的最大内存数(单位: MB)(如果 `MEMUSAGE_ENABLED`为True)。 如果为0，将不发送警告。

##NEWSPIDER_MODULE##

默认：<font color=red>`''`</font>

使用 `genspider` 命令创建新spider的模块。

样例：

	NEWSPIDER_MODULE = 'mybot.spiders_dev'

##RANDOMIZE\_DOWNLOAD\_DELAY##

默认：<font color=red>`True`</font>

如果启用，当从相同的网站获取数据时，Scrapy将会等待一个随机的值 (0.5到1.5之间的一个随机值 * `DOWNLOAD_DELAY`)。

该随机值降低了crawler被检测到(随后被封禁)的机会。某些网站会分析请求， 查找请求之间时间的相似性。

随机的策略与 wget <font color=red>`--random-wait`</font> 选项的策略相同。

若 `	DOWNLOAD_DELAY` 为0(默认值)，该选项将不起作用。

##REACTOR\_THREADPOOL\_MAXSIZE##

默认：<font color=red>`10`</font>

Twisted Reator 线程池线程数的最大数目。这是被Scrapy各种组件使用的常见的多目的线程池。单线程DNS解析器，BlockingFeedStorage， S3FilesStore 等等。如果阻塞IO不足请增大这个值。

##REDIRECT\_MAX\_TIMES##

默认：<font color=red>`20`</font>

定义request允许重定向的最大次数。超过该限制后该request直接返回获取到的结果。 对某些任务我们使用Firefox默认值。

##REDIRECT\_PRIORITY\_ADJUST##

默认：<font color=red>`+2`</font>

应用于：<font color=red>`scrapy.downloadermiddlewares.redirect.RedirectMiddleware`</font>

相对原始请求，调整重定向请求的优先级。

  - 数值为正(默认)意味着更高的优先级
  - 数值为负意味着更低的优先级

##RETRY\_PRIORITY\_ADJUST##

默认：<font color=red>`-1`</font>

应用于：<font color=red>`scrapy.downloadermiddlewares.retry.RetryMiddleware`</font>

相对原始请求，调整重试请求的优先级。

  - 数值为正(默认)意味着更高的优先级
  - 数值为负意味着更低的优先级


##ROBOTSTXT_OBEY##

默认：<font color=red>`False`</font>

应用于：<font color=red>`scrapy.downloadermiddlewares.robotstxt`</font>

如果启用，Scrapy将会遵循robots.txt策略。

<font color=orange>
NOTE：</br>
默认值为<font color=red>`False`</font>是因为历史原因。但是在 <font color=red>`scrapy startproject`</font> 命令生成的 settings.py 文件中，此选项是启用的。
</font>

##SCHEDULER##

默认：<font color=red>`'scrapy.core.scheduler.Scheduler'`</font>

用于爬取的调度器。

##SCHEDULER_DEBUG##

默认：<font color=red>`False`</font>

设置为<font color=red>`True`</font>将会日志记录请求调度器的调试信息。如果请求无法序列化到磁盘上，日志一般只会记录一次。Stats counter（<font color=red>`scheduler/unserializable`</font>）跟踪发生这种情况的次数。

日志中的示例条目：

	1956-01-31 00:00:00+0800 [scrapy.core.scheduler] ERROR: Unable to serialize request:
	<GET http://example.com> - reason: cannot serialize <Request at 0x9a7c7ec>
	(type Request)> - no more unserializable requests will be logged
	(see 'scheduler/unserializable' stats counter)

##SCHEDULER\_DISK\_QUEUE##

默认：<font color=red>`'scrapy.squeues.PickleLifoDiskQueue'`</font>

调度器使用的磁盘队列的类型。其他可用的类型有：
<font color=red>`scrapy.squeues.PickleFifoDiskQueue`</font>，<font color=red>`scrapy.squeues.MarshalFifoDiskQueue`</font><font color=red>`scrapy.squeues.MarshalLifoDiskQueue`</font>

##SCHEDULER\_MEMORY\_QUEUE##

默认：<font color=red>`'scrapy.squeues.LifoMemoryQueue'`</font>

调度器使用的内存队列的类型。其他可用的类型有：
<font color=red>`scrapy.squeues.FifoMemoryQueue`</font>

##SCHEDULER\_PRIORITY\_QUEUE##

默认：<font color=red>`'queuelib.PriorityQueue'`</font>

调度器使用的优先级队列的类型。

##SPIDER_CONTRACTS##

默认：<font color=red>`{}`</font>

保存项目中启用用于测试spider的scrapy contract及其顺序的字典。 更多内容请参考 Spiders Contracts 。

##SPIDER\_CONTRACTS\_BASE##

默认：

	{
	    'scrapy.contracts.default.UrlContract' : 1,
	    'scrapy.contracts.default.ReturnsContract': 2,
	    'scrapy.contracts.default.ScrapesContract': 3,
	}

保存Scrapy默认的的scrapy contract及其顺序的字典。你的项目中，不能修改此设置，而是修改 `SPIDER_CONTRACT`。

你可以在`SPIDER_CONTRACTS`中通过声明它们的类路径为<font color=red>`None`</font>来禁用contracts。例如禁用内置的 <font color=red>`ScrapesContract`</font>，请在 <font color=red>`settings.py`</font>中这样设置：

	SPIDER_CONTRACTS = {
	    'scrapy.contracts.default.ScrapesContract': None,
	}

##SPIDER\_LOADER\_CLASS##

默认：<font color=red>`'scrapy.spiderloader.SpiderLoader'`</font>

这个类将会被用作加载spiders，必须实现 SpiderLoader API。

##SPIDER\_LOADER\_WARN\_ONLY##


默认：<font color=red>`Flase`</font>

默认情况下，当Scrapy尝试从`SPIDER_MODULES`中导入spider类时，当有任何的<font color=red>`ImportError`</font>异常发生时，都会大声宣告失败。但是你可以选择让这个异常沉默，返回一个简单的警告，通过设置这个:<font color=red>`SPIDER_LOADER_WARN_ONLY = True`</font>

<font color=orange>
NOTE：</br>
一些Scrapy命令运行时已经设置这个为 <font color=red>`True`</font>了(即它们只会警告不会失败)，因为实际上它们工作不需要加载spider类：`scrapy runspider`，`scrapy startproject`，`scrapy settings`，`scrapy vesion`</font>

##SPIDER_MIDDLEWARES##

默认：<font color=red>`{}`</font>

保存项目中启用的下载中间件及其顺序的字典。

##SPIDER\_MIDDLEWARES\_BASE##

默认：

	{
	    'scrapy.spidermiddlewares.httperror.HttpErrorMiddleware': 50,
	    'scrapy.spidermiddlewares.offsite.OffsiteMiddleware': 500,
	    'scrapy.spidermiddlewares.referer.RefererMiddleware': 700,
	    'scrapy.spidermiddlewares.urllength.UrlLengthMiddleware': 800,
	    'scrapy.spidermiddlewares.depth.DepthMiddleware': 900,
	}

保存项目中默认启用的spider中间件的字典。 永远不要在项目中修改该设定，而是修改 `SPIDER_MIDDLEWARES`。

##SPIDER_MODULES##

默认：<font color=red>`{}`</font>

Scrapy搜索spider的模块列表

样例:

	SPIDER_MODULES = ['mybot.spiders_prod', 'mybot.spiders_dev']

##STATS_CLASS##

默认：<font color=red>`'scrapy.statscollectors.MemoryStatsCollector'`</font>

收集数据的类。该类必须实现 状态收集器(Stats Collector) API.

##STATSMAILER_RCPTS##


默认：<font color=red>`[]`</font>(空列表)

spider完成爬取后发送Scrapy数据。更多内容请查看 `StatsMailer`。

##TELNETCONSOLE_ENABLED##

默认：<font color=red>`True`</font>

表明 telnet 终端 (及其插件)是否启用的布尔值。

##TELNETCONSOLE_PORT##

默认：<font color=red>`[6023, 6073]`</font>

telnet终端使用的端口范围。如果设置为 <font color=red>`None`</font> 或 <font color=red>`0`</font> ， 则使用动态分配的端口。更多内容请查看 Telnet终端(Telnet Console) 。

##TEMPLATES_DIR##

默认：scrapy模块内部的<font color=red>`templates`</font>

使用 `startproject` 命令创建项目和使用 `genspider` 创建新的spider时查找模板的目录。


项目名称不得与<font color=red>`project`</font>子目录中的自定义文件或目录的名称冲突。

##URLLENGTH_LIMIT##

默认：<font color=red>`2083`</font>

应用于: <font color=red>`spidermiddlewares.urllength`</font>

爬取URL的最大长度。更多关于该设定的默认值信息请查看: http://www.boutell.com/newfaq/misc/urllength.html

##USER_AGENT##

默认：<font color=red>`"Scrapy/VERSION (+https://scrapy.org)"`</font>

爬取的默认User-Agent，除非被覆盖。

#其他地方settings#

  - AJAXCRAWL_ENABLED
  - AUTOTHROTTLE_DEBUG
  - AUTOTHROTTLE_ENABLED
  - AUTOTHROTTLE\_MAX\_DELAY
  - AUTOTHROTTLE\_START\_DELAY
  - AUTOTHROTTLE\_TARGET\_CONCURRENCY
  - CLOSESPIDER_ERRORCOUNT
  - CLOSESPIDER_ITEMCOUNT
  - CLOSESPIDER_PAGECOUNT
  - CLOSESPIDER_TIMEOUT
  - COMMANDS_MODULE
  - COMPRESSION_ENABLED
  - COOKIES_DEBUG
  - COOKIES_ENABLED
  - FEED\_EXPORTERS
  - FEED\_EXPORTERS\_BASE
  - FEED\_EXPORT\_ENCODING
  - FEED\_EXPORT\_FIELDS
  - FEED\_EXPORT\_INDENT
  - FEED_FORMAT
  - FEED_STORAGES
  - FEED\_STORAGES\_BASE
  - FEED\_STORE\_EMPTY
  - FEED_URI
  - FILES_EXPIRES
  - FILES\_RESULT\_FIELD
  - FILES_STORE
  - FILES\_STORE\_S3\_ACL
  - FILES\_URLS\_FIELD
  - GCS_PROJECT_ID
  - HTTPCACHE\_ALWAYS\_STORE
  - HTTPCACHE_DBM_MODULE
  - HTTPCACHE_DIR
  - HTTPCACHE_ENABLED
  - HTTPCACHE\_EXPIRATION\_SECS
  - HTTPCACHE_GZIP
  - HTTPCACHE\_IGNORE\_HTTP\_CODES
  - HTTPCACHE\_IGNORE\_MISSING
  - HTTPCACHE\_IGNORE\_RESPONSE\_CACHE\_CONTROLS
  - HTTPCACHE\_IGNORE\_SCHEMES
  - HTTPCACHE_POLICY
  - HTTPCACHE_STORAGE
  - HTTPERROR\_ALLOWED\_CODES
  - HTTPERROR\_ALLOW\_ALL
  - HTTPPROXY\_AUTH\_ENCODING
  - HTTPPROXY_ENABLED
  - IMAGES_EXPIRES
  - IMAGES_MIN_HEIGHT
  - IMAGES_MIN_WIDTH
  - IMAGES\_RESULT\_FIELD
  - IMAGES_STORE
  - IMAGES\_STORE\_S3\_ACL
  - IMAGES_THUMBS
  - IMAGES\_URLS\_FIELD
  - MAIL_FROM
  - MAIL_HOST
  - MAIL_PASS
  - MAIL_PORT
  - MAIL_SSL
  - MAIL_TLS
  - MAIL_USER
  - MEDIA\_ALLOW\_REDIRECTS
  - METAREFRESH_ENABLED
  - METAREFRESH_MAXDELAY
  - REDIRECT_ENABLED
  - REDIRECT_MAX_TIMES
  - REFERER_ENABLED
  - REFERRER_POLICY
  - RETRY_ENABLED
  - RETRY_HTTP_CODES
  - RETRY_TIMES
  - TELNETCONSOLE_HOST
  - TELNETCONSOLE_PORT