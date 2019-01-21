#下载中间件(Downloader Middleware)#

下载器中间件是介于Scrapy的request/response处理的钩子框架。 是用于全局修改Scrapy request和response的一个轻量、底层的系统。

##激活下载器中间件(Activating a downloader middleware)##

要激活下载器中间件组件，将其加入到 `DOWNLOADER_MIDDLEWARES` 设置中。 该设置是一个字典(dict)，键为中间件类的路径，值为其中间件的顺序(order)。

这里是一个例子:

	DOWNLOADER_MIDDLEWARES = {
	    'myproject.middlewares.CustomDownloaderMiddleware': 543,
	}

`DOWNLOADER_MIDDLEWARES` 设置会与Scrapy定义的 `DOWNLOADER_MIDDLEWARES_BASE` 设置合并(但不是覆盖)， 而后根据顺序(order)进行排序，最后得到启用中间件的有序列表: 第一个中间件是最靠近引擎的，最后一个中间件是最靠近下载器的。换句话说，每个中间件的 `process_request()` 方法会按照中间件的递增顺序(100, 200, 300...) 挨个调用，`process_response()` 方法按中间件的递减顺序调用。

关于如何分配中间件的顺序请查看 `DOWNLOADER_MIDDLEWARES_BASE` 设置，而后根据您想要放置中间件的位置选择一个值。 由于每个中间件执行不同的动作，您的中间件可能会依赖于之前(或者之后)执行的中间件，因此顺序是很重要的。

如果您想禁止内置的(在 `DOWNLOADER_MIDDLEWARES_BASE` 中设置并默认启用的)中间件， 您必须在项目的 `DOWNLOADER_MIDDLEWARES` 设置中定义该中间件，并将其值赋为 <font color=red>`None`</font> 。 例如，如果您想要关闭user-agent中间件:

	DOWNLOADER_MIDDLEWARES = {
	    'myproject.middlewares.CustomDownloaderMiddleware': 543,
	    'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': None,
	}

最后，请注意，有些中间件需要通过特定的设置来启用。更多内容请查看相关中间件文档。

##编写您自己的下载器中间件(Writing your own downloader middleware)##

每个中间件组件是一个定义了以下一个或多个方法的Python类:

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.downloadermiddlewares.DownloaderMiddleware
</td></tr></table>

**process_request(request, spider)**

当每个request通过下载中间件时，该方法被调用。

`process_request()` 必须返回其中之一: 返回 <font color=red>`None`</font> 、返回一个 <font color=red>`Response`</font> 对象、返回一个 <font color=red>`Request`</font> 对象或raise <font color=red>`IgnoreRequest`</font> 。

如果其返回 <font color=red>`None`</font> ，Scrapy将继续处理该request，执行其他的中间件的相应方法，直到合适的下载器处理函数(download handler)被调用， 该request被执行(其response被下载)。

如果其返回 <font color=red>`Response`</font> 对象，Scrapy将不会调用 任何 其他的 `process_request()` 或 `process_exception()` 方法，或相应地下载函数； 其将返回该 <font color=red>`Response`</font>。 已安装的中间件的 `process_response()` 方法则会在每个response返回时被调用。

如果其返回 <font color=red>`Request`</font> 对象，Scrapy则停止调用 `process_request`方法并重新调度返回的request。当新返回的request被执行后， 相应地中间件链将会根据下载的response被调用。

如果其raise一个 <font color=red>`IgnoreRequest`</font> 异常，则安装的下载中间件的 `process_exception()` 方法会被调用。如果没有任何一个方法处理该异常， 则request的`errback(Request.errback)`方法会被调用。如果没有代码处理抛出的异常， 则该异常被忽略且不记录(不同于其他异常那样)。

&nbsp;&nbsp;&nbsp;**参数**：
  
 - **request**(`Request`对象) - 处理的request
 - **spider**(`Spider`对象) - 该request对应的spider

**process_response(request, response, spider)**

`process_request()` 必须返回以下之一: 返回一个 <font color=red>`Response`</font> 对象、 返回一个 <font color=red>`Request`</font> 对象或raise一个 <font color=red>`IgnoreRequest`</font> 异常。

如果其返回一个 <font color=red>`Response`</font> (可以与传入的response相同，也可以是全新的对象)， 该response会被在链中的其他中间件的 `process_response()` 方法处理。

如果其返回一个 <font color=red>`Request`</font> 对象，则中间件链停止， 返回的request会被重新调度下载。处理类似于` process_request()` 返回request所做的那样。

如果其抛出一个 <font color=red>`IgnoreRequest`</font> 异常，则调用request的`errback(Request.errback)`。 如果没有代码处理抛出的异常，则该异常被忽略且不记录(不同于其他异常那样)。

&nbsp;&nbsp;&nbsp;**参数**：
  
 - **request**(`Request`对象) - response所对应的request
 - **response**(`Response`对象) - 被处理的response
 - **spider**(`Spider`对象) - 该response对应的spider

**process_exception(request, exception, spider)**

当下载处理器(download handler)或 `process_request()` (下载中间件)抛出异常(包括 `IgnoreRequest` 异常)时， Scrapy调用 `process_exception() `。

`process_exception() `应该返回以下之一: 返回 <font color=red>`None`</font> 、 一个 <font color=red>`Response`</font> 对象、或者一个 <font color=red>`Request`</font> 对象。

如果其返回 <font color=red>`None`</font> ，Scrapy将会继续处理该异常，接着调用已安装的其他中间件的 `process_exception()` 方法，直到所有中间件都被调用完毕，则调用默认的异常处理。

如果其返回一个 <font color=red>`Response`</font> 对象，则已安装的中间件链的 `process_response()` 方法被调用。Scrapy将不会调用任何其他中间件的 `process_exception()` 方法。

如果其返回一个 <font color=red>`Request`</font> 对象， 则返回的request将会被重新调用下载。这将停止中间件的 `process_exception()`方法执行，就如返回一个response的那样。

&nbsp;&nbsp;&nbsp;**参数**：

   - **request** (是 `Request` 对象) – 产生异常的request
   - **exception** (`Exception` 对象) – 抛出的异常
   - **spider** (`Spider` 对象) – request对应的spider

**from_crawler(cls, crawler)**

如果存在，这个类方法被调用从`Crawler`中创建一个中间件实例。它必须返回一个新的中间件实例。Crawler对象提供访问所有Scrapy核心组件，例如settings和signals；这是middleware去访问这些组件和使中间件的功能与Scrapy关联的一种方式。

&nbsp;&nbsp;&nbsp;**参数**：

  - **crawler**('Crawler'对象) - 使用中间件的crawler。

##内置的下载中间件参考(Built-in downloader middleware reference)##

本页面介绍了Scrapy自带的所有下载中间件。关于如何使用及编写您自己的中间件，请参考 downloader middleware usage guide。


关于默认启用的中间件列表(及其顺序)请参考 `DOWNLOADER_MIDDLEWARES_BASE` 设置。

###CookiesMiddleware###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.downloadermiddlewares.cookies.CookiesMiddleware
</td></tr></table>

该中间件使得爬取需要cookie(例如使用session)的网站成为了可能。 其追踪了web server发送的cookie，并在之后的request中发送回去， 就如浏览器所做的那样。

The following settings can be used to configure the cookie middleware:

  - `COOKIES_ENABLED`
  - `COOKIES_DEBUG`

#####Multiple cookie sessions per spider#####

Scrapy通过使用 `cookiejar` Request meta key来支持单spider追踪多cookie session。 默认情况下其使用一个cookie jar(session)，不过您可以传递一个标示符来使用多个。

例如：

	for i, url in enumerate(urls):
	    yield scrapy.Request(url, meta={'cookiejar': i},
	        callback=self.parse_page)

需要注意的是 `cookiejar` meta key不是”黏性的(sticky)”。 您需要在之后的request请求中接着传递。例如：

	def parse_page(self, response):
	    # do some processing
	    return scrapy.Request("http://www.example.com/otherpage",
	        meta={'cookiejar': response.meta['cookiejar']},
	        callback=self.parse_other_page)

#####COOKIES_ENABLED#####

默认：<font color=red>`True`</font>

是否启用cookies middleware。如果关闭，没有cookies发送给web server。

注意尽管 `COOKIES_ENABLED` 设置了值，但是如果 <font color=red>`Request.meta['dont_merge_cookies']`</font>的值为 <font color=red>`True`</font>，请求的cookies将不会发送给web server，`Response` 收到的cookies 也不会与已存在的cookies合并。


#####COOKIES_DEBUG#####

默认：<font color=red>`True`</font>

如果启用，Scrapy将记录所有在request(<font color=red>`Cookie`</font> 请求头)发送的cookies及response接收到的cookies(<font color=red>`Set-Cookie`</font> 接收头)。

下边是启用 `COOKIES_DEBUG` 的记录的样例:

	2011-04-06 14:35:10-0300 [scrapy.core.engine] INFO: Spider opened
	2011-04-06 14:35:10-0300 [scrapy.downloadermiddlewares.cookies] DEBUG: Sending cookies to: <GET http://www.diningcity.com/netherlands/index.html>
	        Cookie: clientlanguage_nl=en_EN
	2011-04-06 14:35:14-0300 [scrapy.downloadermiddlewares.cookies] DEBUG: Received cookies from: <200 http://www.diningcity.com/netherlands/index.html>
	        Set-Cookie: JSESSIONID=B~FA4DC0C496C8762AE4F1A620EAB34F38; Path=/
	        Set-Cookie: ip_isocode=US
	        Set-Cookie: clientlanguage_nl=en_EN; Expires=Thu, 07-Apr-2011 21:21:34 GMT; Path=/
	2011-04-06 14:49:50-0300 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.diningcity.com/netherlands/index.html> (referer: None)
	[...]

###DefaultHeadersMiddleware###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware
</td></tr></table>

该中间件使用 `DEFAULT_REQUEST_HEADERS` 设置指定值为所有request设置默认request header。

###DownloadTimeoutMiddleware###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware
</td></tr></table>

该中间件通过使用 `DOWNLOAD_TIMEOUT`设置或者 `download_timeout` spider属性指定的值为请求设置下载超时时间。

<font color=orange>
注意：</br>你也可以通过 `download_timeout` Request.meta key为每个请求设置下载超时时间；这个在 `DownloadTimeoutMiddleware` 被禁用的情况下也能使用。
</font>

###HttpAuthMiddleware###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware
</td></tr></table>

该中间件完成某些使用 Basic access authentication (或者叫HTTP认证)的spider生成的请求的认证过程。

在spider中启用HTTP认证，请设置spider的 <font color=red>`http_user`</font> 及 <font color=red>`http_pass`</font> 属性。

样例：

	from scrapy.spiders import CrawlSpider
	
	class SomeIntranetSiteSpider(CrawlSpider):
	
	    http_user = 'someuser'
	    http_pass = 'somepass'
	    name = 'intranet.example.com'
	
	    # .. rest of the spider code omitted ...


###HttpCacheMiddleware###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.downloadermiddlewares.httpcache.HttpCacheMiddleware
</td></tr></table>

该中间件为所有HTTP request及response提供了低级(low-level)缓存支持。 其由cache存储后端及cache策略组成。

Scrapy提供了三种HTTP缓存存储后端：

  - Filesystem storage backend (default)
  - DBM storage backend
  - LevelDB storage backend

您可以使用 `HTTPCACHE_STORAGE` 设定来修改HTTP缓存存储后端。 您也可以实现您自己的存储后端。

Scrapy提供了两种了缓存策略:

  - RFC2616 policy
  - Dummy policy (default)

您可以使用 `HTTPCACHE_POLICY` 设定来修改HTTP缓存存储后端。 您也可以实现您自己的存储策略。

你可以使用 `dont_cache` meta键设置为 `True` 来避免在每个策略上都缓存响应。

#####Dummy policy (default)#####

该策略不考虑任何HTTP Cache-Control指令。每个request及其对应的response都被缓存。 当相同的request发生时，其不发送任何数据，直接返回response。

Dummpy策略对于测试spider十分有用。其能使spider运行更快(不需要每次等待下载完成)， 同时在没有网络连接时也能测试。其目的是为了能够回放spider的运行过程， 使之与之前的运行过程一模一样 。

使用这个策略请设置：

  - `HTTPCACHE_POLICY` 为 <font color=red>`scrapy.extensions.httpcache.DummyPolicy`</font>


#####RFC2616 policy#####

该策略提供了符合RFC2616的HTTP缓存，例如符合HTTP Cache-Control， 针对生产环境并且应用在持续性运行环境所设置。该策略能避免下载未修改的数据(来节省带宽，提高爬取速度)。

实现了：

  - 当 no-store cache-control指令设置时不存储response/request。
  - 当 no-cache cache-control指定设置时不从cache中提取response，即使response为最新。
  - 根据 max-age cache-control指令中计算保存时间(freshness lifetime)。
  - 根据 Expires 指令来计算保存时间(freshness lifetime)。
  - 根据response包头的 Last-Modified 指令来计算保存时间(freshness lifetime)(Firefox使用的启发式算法)。
  - 根据response包头的 Age 计算当前年龄(current age)
  - 根据 Date 计算当前年龄(current age)
  - 根据response包头的 Last-Modified 验证老旧的response。
  - 根据response包头的 ETag 验证老旧的response。
  - 为接收到的response设置缺失的 Date 字段。
  - 请求中支持 `max-stale` cache-control指令。	</br>这个给予spider配置为完全的RFC2616缓存策略，避免在逐个请求的基础上重新验证，同时保持符合HTTP规范。</br>例如：</br>添加Cache-Control：max-stale = 600到Request请求头上，以接受超过其到期时间不超过600秒的响应。


目前仍然缺失：

  - Pragma: no-cache 支持  `https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9.1`
  - Vary 字段支持 `https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.6`
  - 当update或delete之后失效相应的response ` https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.10`
  - ... 以及其他可能缺失的特性 ..
 
使用这个策略，设置：

  - `HTTPCACHE_POLICY` 为 <font color=red>`scrapy.extensions.httpcache.RFC2616Policy`</font>

#####Filesystem storage backend (default)#####

文件系统存储后端可以用于HTTP缓存中间件。

使用该存储端，设置:

  - `HTTPCACHE_STORAGE` 为 <font color=red>`scrapy.contrib.httpcache.FilesystemCacheStorage`</font>

每个request/response组存储在不同的目录中，包含下列文件：

  - <font color=red>`request_body`</font> - the plain request body
  - <font color=red>`request_headers`</font> - the request headers (原始HTTP格式)
  - <font color=red>`response_body`</font> - the plain response body
  - <font color=red>`response_headers`</font> - the request headers (原始HTTP格式)
  - <font color=red>`meta`</font> - 以Python <font color=red>`repr()`</font> 格式(grep-friendly格式)存储的该缓存资源的一些元数据。
  - <font color=red>`pickled_meta`</font> - 与 <font color=red>`meta`</font> 相同的元数据，不过使用pickle来获得更高效的反序列化性能。

目录的名称与request的指纹(参考 <font color=red>`scrapy.utils.request.fingerprint`</font>)有关，而二级目录是为了避免在同一文件夹下有太多文件 (这在很多文件系统中是十分低效的)。目录的例子：

	/path/to/cache/dir/example.com/72/72811f648e718090f041317756c03adb0ada46c7

#####DBM storage backend#####

同时也有 DBM 存储后端可以用于HTTP缓存中间件。

默认情况下，其采用 anydbm 模块，不过您也可以通过 `HTTPCACHE_DBM_MODULE` 设置进行修改。

使用该存储端，设置:

  - `HTTPCACHE_STORAGE` 为 <font color=red>`scrapy.contrib.httpcache.DbmCacheStorage`</font>


#####LevelDB storage backend#####


LevelDB 存储后端可以用于HTTP缓存中间件。

建议不要将此后端用于开发，因为只有一个进程可以同时访问LevelDB数据库,
因此你不能并行地运行爬虫和通过shell打开同一个spider。

使用该存储端，设置：

  - 设置 `HTTPCACHE_STORAGE` 为  <font color=red>`scrapy.extensions.httpcache.LeveldbCacheStorage`</font>
  - 安装 LevelDB python bindings： <font color=red>`pip install leveldb`</font>

###HTTPCache middleware settings###

`HttpCacheMiddleware` 可以通过以下设置进行配置:

#####HTTPCACHE_ENABLED#####

默认：<font color=red>`False`</font>

HTTP缓存是否开启。

在 0.11 版更改: 在0.11版本前，是使用 `HTTPCACHE_DIR` 来开启缓存。

#####HTTPCACHE_EXPIRATION\_SECS#####

默认：<font color=red>`0`</font>


缓存的request的超时时间，单位秒。

超过这个时间的缓存request将会被重新下载。如果为0，则缓存的request将永远不会超时。

在 0.11 版更改: 在0.11版本前，0的意义是缓存的request永远超时。


#####HTTPCACHE_DIR#####

默认：<font color=red>`'httpcache'`</font>

存储(低级的)HTTP缓存的目录。如果为空，则HTTP缓存将会被关闭。 如果为相对目录，则相对于项目数据目录(project data dir)


#####HTTPCACHE\_IGNORE_HTTP\_CODES#####

默认：<font color=red>`[]`</font>


不缓存列表中的HTTP状态码(code)的响应。


#####HTTPCACHE_IGNORE\_MISSING#####

默认：<font color=red>`False`</font>

如果启用，在缓存中没找到的request将会被忽略，而不是重新下载。

#####HTTPCACHE_IGNORE\_SCHEMES#####

默认：<font color=red>`['file']`</font>

不缓存这些URI schemel(://前面的字段，例如http://前面的字段是http)的响应。

#####HTTPCACHE_STORAGE#####

默认：<font color=red>`'scrapy.extensions.httpcache.FilesystemCacheStorage'`</font>

实现缓存存储后端的类。

#####HTTPCACHE_DBM\_MODULE#####

默认：<font color=red>`'anydbm'`</font>

使用 DBM存储后端 的数据库模块。 该设置是DBM后端特定的。

#####HTTPCACHE_POLICY#####

默认：<font color=red>`'scrapy.extensions.httpcache.DummyPolicy'`</font>

实现缓存策略的类。

#####HTTPCACHE_GZIP#####

默认：<font color=red>`False`</font>

如果启用，将会压缩所有的缓存数据为gzip格式。这个设置Filesystem后端特定的。

#####HTTPCACHE_ALWAYS\_STORE#####

默认：<font color=red>`False`</font>

如果设置了，将无条件的缓存网页。

spider可能希望在缓存中有所有可用的响应，以后使用 Cache-Control:max-stale。DummyPolicy缓存所有的响应，但是不会重新验证它们，有时需要更加细微的策略。

这个策略仍然遵循响应中的 Cache-Control:no-store指令。如果你不想这样，在传入缓存中间件之前过滤掉响应中的Cache-Control:no-store头。

#####HTTPCACHE\_IGNORE\_RESPONSE_CACHE\_CONTROLS#####

默认：<font color=red>`[]`</font>

响应的Cache-Control指令值为列表所列的，将会忽略这些Cache-Control。

网站经常设置为 'no-store'，'no-cache'， 'must-revalidate'等等，但是如果spider遵循这些指令的话，将对爬虫不友好。这个设置允许爬虫有选择的忽略对被爬取的网站不重要的Cache-Control指令。

我们假设spider不会在请求中发出Cache-Control指令，除非它实际需要它们，因此请求中的指令不会被过滤。


###HttpCompressionMiddleware###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware
</td></tr></table>

该中间件提供了对压缩(gzip, deflate)数据的支持。如果安装了brotlipy，此中间件还支持解码brotli-compressed响应。

###设置(HttpCompressionMiddleware Settings)###

#####COMPRESSION_ENABLED#####

默认：<font color=red>`True`</font>

Compression Middleware(压缩中间件)是否开启。

###HttpProxyMiddleware###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware
</td></tr></table>

该中间件提供了对request设置HTTP代理的支持。您可以通过在 `Request` 对象中设置 <font color=red>`proxy`</font> 元数据来开启代理。

类似于Python标准库模块 urllib 及 urllib2 ，其使用了下列环境变量:

  - <font color=red>`http_proxy`</font>
  - <font color=red>`https_proxy`</font>
  - <font color=red>`no_proxy`</font>

你可以为每个请求设置meta key <font color=red>`proxy`</font>，设置的值就像 <font color=red>`http://some_proxy_server:port`</font>
或者 <font color=red>`http://username:password@some_proxy_server:port`</font> 这样。记住这个值比环境变量 <font color=red>`http_proxy`</font> / <font color=red>`https_proxy`</font> 优先级高，也会忽略 <font color=red>`no_proxy`</font> 环境变量。


###HttpProxyMiddleware###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.downloadermiddlewares.redirect.RedirectMiddleware
</td></tr></table>

该中间件根据response的状态处理重定向的request。

通过该中间件的(被重定向的)request的url可以通过 `Request.meta` 的 <font color=red>`redirect_urls`</font> 键找到。

`RedirectMiddleware` 可以通过下列设置进行配置(更多内容请参考设置文档)：

  - `REDIRECT_ENABLED`
  - `REDIRECT_MAX_TIMES`

如果 `Request.meta` 包含 <font color=red>`dont_redirect`</font> 键，并且值为True，则该request将会被此中间件忽略。

如果你想要在spider中处理一些重定向响应码，你可以指定 <font color=red>`handle_httpstatus_list`</font> spider属性。

例如，你想要 redirect middleware 忽略301和302响应码(并把响应传递给spider)，你可以这样：

	class MySpider(CrawlSpider):
	    handle_httpstatus_list = [301, 302]

`Request.meta`key 为 <font color=red>`handle_httpstatus_list`</font> 也能为单个请求指定响应码。如果你想允许所有的响应码你也可以设置meta key <font color=red>`handle_httpstatus_all`</font> 为 <font color=red>`True`</font>。

###(设置)RedirectMiddleware settings###

#####REDIRECT_ENABLED#####

默认：<font color=red>`True`</font>

是否启用Redirect中间件。

#####REDIRECT_MAX\_TIMES#####

默认：<font color=red>`20`</font>

单个request被重定向的最大次数。

###MetaRefreshMiddleware###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware
</td></tr></table>

该中间件根据meta-refresh html标签处理request重定向。

`MetaRefreshMiddleware` 可以通过以下设定进行配置 (更多内容请参考设置文档)。

  - METAREFRESH_ENABLED
  - METAREFRESH_MAXDELAY

该中间件遵循 `RedirectMiddleware` 描述的 `REDIRECT_MAX_TIMES` 设定，`dont_redirect` 及 `redirect_urls` meta key。

###设置（MetaRefreshMiddleware settings)###

#####METAREFRESH_ENABLED#####

默认：<font color=red>`True`</font>

Meta Refresh中间件是否启用

#####METAREFRESH_MAXDELAY#####

默认：<font color=red>`100`</font>

跟进重定向的最大 meta-refresh 延迟(单位:秒)。一些网站使用meta-refresh重定向到一个会话的过期网页，因此我们限制自动重定向的最大延迟。

###RetryMiddleware###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.downloadermiddlewares.retry.RetryMiddleware
</td></tr></table>

该中间件将重试可能由于临时的问题，例如连接超时或者HTTP 500错误导致失败的页面。

爬取进程会收集失败的页面并在最后，spider爬取完所有正常(不失败)的页面后重新调度。 一旦没有更多需要重试的失败页面，该中间件将会发送一个信号(retry_complete)， 其他插件可以监听该信号。

The `RetryMiddleware` can be configured through the following settings (see the settings documentation for more info):

  - `RETRY_ENABLED`
  - `RETRY_TIMES`
  - `RETRY_HTTP_CODES`

如果 `Request.meta` 包含 <font color=red>`dont_retry`</font> 键，并且值为True，则该request将会被此中间件忽略。

###设置(RetryMiddleware Settings)###

#####RETRY_ENABLED#####

默认：<font color=red>`True`</font>

Retry Middleware是否启用。

###RETRY_TIMES###

默认：<font color=red>`2`</font>

包括第一次下载，最多的重试次数。

重试的最大次数可以使用 `Request.meta` 的 `max_retry_times` 属性为每个请求指定。当被初始化的时候，`max_retry_times` 比 `RETRY_TIMES` 的优先级高。

#####RETRY\_HTTP_CODES#####

默认：<font color=red>`[500, 502, 503, 504, 408]`</font>

重试的response 返回值(code)。其他错误(DNS查找问题、连接失败及其他)则一定会进行重试。

某些情况下，你想要把400加到 `RETRY_HTTP_CODES` 因为它通常是作为指示服务器过载的代码。默认情况下不包括它，因为HTTP规范说明了这一点。

###RobotsTxtMiddleware###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.downloadermiddlewares.robotstxt.RobotsTxtMiddleware
</td></tr></table>

该中间件过滤所有robots.txt eclusion standard中禁止的request。

确认该中间件及 `ROBOTSTXT_OBEY` 设置被启用以确保Scrapy尊重robots.txt。

如果 `Request.meta` 有 <font color=red>`dont_obey_robotstxt`</font> 键，并设置为True，请求将会被忽略这个中间件，即使 `ROBOTSTXT_OBEY` 启用。

###DownloaderStats###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.downloadermiddlewares.stats.DownloaderStats
</td></tr></table>

保存所有通过的request、response及exception的中间件。

您必须启用 `DOWNLOADER_STATS` 来启用该中间件。

###UserAgentMiddleware###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.downloadermiddlewares.useragent.UserAgentMiddleware
</td></tr></table>

用于覆盖spider的默认user agent的中间件。

要使得spider能覆盖默认的user agent，其 user_agent 属性必须被设置。


###AjaxCrawlMiddleware###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.downloadermiddlewares.ajaxcrawl.AjaxCrawlMiddleware
</td></tr></table>

根据meta-fragment html标签查找 ‘AJAX可爬取’ 页面的中间件。查看 https://developers.google.com/webmasters/ajax-crawling/docs/getting-started 来获得更多内容。

<font color=orange>
注解：</br>
即使没有启用该中间件，Scrapy仍能查找类似于 'http://example.com/!#foo=bar' 这样的’AJAX可爬取’页面。 AjaxCrawlMiddleware是针对不具有 '!#' 的URL，通常发生在’index’或者’main’页面中。

</font>

###AjaxCrawlMiddleware设置###

#####AJAXCRAWL_ENABLED#####

默认：<font color=red>`False`</font>

AjaxCrawlMiddleware是否启用。您可能需要针对 通用爬虫 启用该中间件。

###HttpProxyMiddleware settings###

#####HTTPPROXY_ENABLED#####

默认：<font color=red>`True`</font>

`HttpProxyMiddleware`是否启用

#####HTTPPROXY_AUTH\_ENCODING#####


默认：<font color=red>`"latin-1"`</font>

`HttpProxyMiddleware` proxy 认证的默认编码。