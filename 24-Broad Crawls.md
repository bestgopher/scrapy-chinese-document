#通用爬虫(Broad Crawls)#

Scrapy默认对特定爬取进行优化。这些站点一般被一个单独的Scrapy spider进行处理， 不过这并不是必须或要求的(例如，也有通用的爬虫能处理任何给定的站点)。

除了这种爬取完某个站点或没有更多请求就停止的”专注的爬虫”，还有一种通用的爬取类型，其能爬取大量(甚至是无限)的网站， 仅仅受限于时间或其他的限制。 这种爬虫叫做”通用爬虫(broad crawls)”，一般用于搜索引擎。

通用爬虫一般有以下通用特性:

  - 其爬取大量(一般来说是无限)的网站而不是特定的一些网站。
  - 其不会将整个网站都爬取完毕，因为这十分不实际(或者说是不可能)完成的。相反，其会限制爬取的时间及数量。
  - 其在逻辑上十分简单(相较于具有很多提取规则的复杂的spider)，数据会在另外的阶段进行后处理(post-processed)
  - 其并行爬取大量网站以避免被某个网站的限制所限制爬取的速度(为表示尊重，每个站点爬取速度很慢但同时爬取很多站点)。

正如上面所述，Scrapy默认设置是对特定爬虫做了优化，而不是通用爬虫。不过， 鉴于其使用了异步架构，Scrapy对通用爬虫也十分适用。 本篇文章总结了一些将Scrapy作为通用爬虫所需要的技巧， 以及相应针对通用爬虫的Scrapy设定的一些建议。

##增加并发(Increase concurrency)##

并发是指同时处理的request的数量。其有全局限制和局部(每个网站)的限制。

Scrapy默认的全局并发限制对同时爬取大量网站的情况并不适用，因此您需要增加这个值。 增加多少取决于您的爬虫能占用多少CPU。 一般开始可以设置为 100 。不过最好的方式是做一些测试，获得Scrapy进程占取CPU与并发数的关系。 为了优化性能，您应该选择一个能使CPU占用率在80%-90%的并发数。

增加全局并发数:

	CONCURRENT_REQUESTS = 100

##增加Twisted的IO线程池最大尺寸(Increase Twisted IO thread pool maximum size)##

目前，Scrapy使用线程池以阻塞的方式解析DNS。在一个更高的并发下爬取会很慢，甚至DNS解析超时。可行的解决方法是增加处理DNS查询的线程数量。DNS队列将会处理更快，以便加速建立连接和爬取整体。

增加线程池尺寸：

	REACTOR_THREADPOOL_MAXSIZE = 20


##(建立自己的DNS)Setup your own DNS##

如果你有多个爬取进程，一个集中的DNS，在DNS服务器上它可能就像像DoS攻击一样造成全部的网络变慢，设置阻塞你的机器。为了避免这种情况，请设置您自己的具有本地缓存​​的DNS服务器，以及一些大型DNS（如OpenDNS或Verizon）的上游。

##降低log级别(Reduce log level)##

当进行通用爬取时，一般您所注意的仅仅是爬取的速率以及遇到的错误。 Scrapy使用 <font color=red>`INFO`</font> log级别来报告这些信息。为了减少CPU使用率(及记录log存储的要求), 在生产环境中进行通用爬取时您不应该使用 <font color=red>`DEBUG`</font> log级别。 不过在开发的时候使用 <font color=red>`DEBUG`</font> 应该还能接受。

设置Log级别:

	LOG_LEVEL = 'INFO'

##禁止cookies(Disable cookies)##

除非您 *真的* 需要，否则请禁止cookies。在进行通用爬取时cookies并不需要， (搜索引擎则忽略cookies)。禁止cookies能减少CPU使用率及Scrapy爬虫在内存中记录的踪迹，提高性能。

禁止cookies:

	COOKIES_ENABLED = False

##禁止重试(Disable retries)##

对失败的HTTP请求进行重试会减慢爬取的效率，尤其是当站点响应很慢(甚至失败)时， 访问这样的站点会造成超时并重试多次。这是不必要的，同时也占用了爬虫爬取其他站点的能力。

禁止重试:

	RETRY_ENABLED = False

##减小下载超时(Reduce download timeout)##

如果您对一个非常慢的连接进行爬取(一般对通用爬虫来说并不重要)， 减小下载超时能让卡住的连接能被快速的放弃并解放处理其他站点的能力。

减小下载超时:

	DOWNLOAD_TIMEOUT = 15

##禁止重定向(Disable redirects)##

除非您对跟进重定向感兴趣，否则请考虑关闭重定向。 当进行通用爬取时，一般的做法是保存重定向的地址，并在之后的爬取进行解析。 这保证了每批爬取的request数目在一定的数量， 否则重定向循环可能会导致爬虫在某个站点耗费过多资源。

关闭重定向:

	REDIRECT_ENABLED = False

##启用 “Ajax Crawlable Pages” 爬取(Enable crawling of “Ajax Crawlable Pages”)##

有些站点(基于2013年的经验数据，之多有1%)声明其为 ajax crawlable 。 这意味着该网站提供了原本只有ajax获取到的数据的纯HTML版本。 网站通过两种方法声明:

  1. 在url中使用 <font color=red>`#!`</font> - 这是默认的方式;
  2. 使用特殊的meta标签 - 这在"main", "index"页面中使用。

Scrapy自动解决(1)；解决(2)您需要启用 AjaxCrawlMiddleware:

	AJAXCRAWL_ENABLED = True

通用爬取经常抓取大量的 “index” 页面； AjaxCrawlMiddleware能帮助您正确地爬取。 由于有些性能问题，且对于特定爬虫没有什么意义，该中间默认关闭。