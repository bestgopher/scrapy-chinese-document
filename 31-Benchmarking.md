#Benchmarking#

Scrapy提供了一个简单的性能测试工具。其创建了一个本地HTTP服务器，并以最大可能的速度进行爬取。 该测试性能工具目的是测试Scrapy在您的硬件上的效率，来获得一个基本的底线用于对比。 其使用了一个简单的spider，仅跟进链接，不做任何处理。

运行:

	scrapy bench

您能看到类似的输出:

	2016-12-16 21:18:48 [scrapy.utils.log] INFO: Scrapy 1.2.2 started (bot: quotesbot)
	2016-12-16 21:18:48 [scrapy.utils.log] INFO: Overridden settings: {'CLOSESPIDER_TIMEOUT': 10, 'ROBOTSTXT_OBEY': True, 'SPIDER_MODULES': ['quotesbot.spiders'], 'LOGSTATS_INTERVAL': 1, 'BOT_NAME': 'quotesbot', 'LOG_LEVEL': 'INFO', 'NEWSPIDER_MODULE': 'quotesbot.spiders'}
	2016-12-16 21:18:49 [scrapy.middleware] INFO: Enabled extensions:
	['scrapy.extensions.closespider.CloseSpider',
	 'scrapy.extensions.logstats.LogStats',
	 'scrapy.extensions.telnet.TelnetConsole',
	 'scrapy.extensions.corestats.CoreStats']
	2016-12-16 21:18:49 [scrapy.middleware] INFO: Enabled downloader middlewares:
	['scrapy.downloadermiddlewares.robotstxt.RobotsTxtMiddleware',
	 'scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware',
	 'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware',
	 'scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware',
	 'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware',
	 'scrapy.downloadermiddlewares.retry.RetryMiddleware',
	 'scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware',
	 'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware',
	 'scrapy.downloadermiddlewares.redirect.RedirectMiddleware',
	 'scrapy.downloadermiddlewares.cookies.CookiesMiddleware',
	 'scrapy.downloadermiddlewares.stats.DownloaderStats']
	2016-12-16 21:18:49 [scrapy.middleware] INFO: Enabled spider middlewares:
	['scrapy.spidermiddlewares.httperror.HttpErrorMiddleware',
	 'scrapy.spidermiddlewares.offsite.OffsiteMiddleware',
	 'scrapy.spidermiddlewares.referer.RefererMiddleware',
	 'scrapy.spidermiddlewares.urllength.UrlLengthMiddleware',
	 'scrapy.spidermiddlewares.depth.DepthMiddleware']
	2016-12-16 21:18:49 [scrapy.middleware] INFO: Enabled item pipelines:
	[]
	2016-12-16 21:18:49 [scrapy.core.engine] INFO: Spider opened
	2016-12-16 21:18:49 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
	2016-12-16 21:18:50 [scrapy.extensions.logstats] INFO: Crawled 70 pages (at 4200 pages/min), scraped 0 items (at 0 items/min)
	2016-12-16 21:18:51 [scrapy.extensions.logstats] INFO: Crawled 134 pages (at 3840 pages/min), scraped 0 items (at 0 items/min)
	2016-12-16 21:18:52 [scrapy.extensions.logstats] INFO: Crawled 198 pages (at 3840 pages/min), scraped 0 items (at 0 items/min)
	2016-12-16 21:18:53 [scrapy.extensions.logstats] INFO: Crawled 254 pages (at 3360 pages/min), scraped 0 items (at 0 items/min)
	2016-12-16 21:18:54 [scrapy.extensions.logstats] INFO: Crawled 302 pages (at 2880 pages/min), scraped 0 items (at 0 items/min)
	2016-12-16 21:18:55 [scrapy.extensions.logstats] INFO: Crawled 358 pages (at 3360 pages/min), scraped 0 items (at 0 items/min)
	2016-12-16 21:18:56 [scrapy.extensions.logstats] INFO: Crawled 406 pages (at 2880 pages/min), scraped 0 items (at 0 items/min)
	2016-12-16 21:18:57 [scrapy.extensions.logstats] INFO: Crawled 438 pages (at 1920 pages/min), scraped 0 items (at 0 items/min)
	2016-12-16 21:18:58 [scrapy.extensions.logstats] INFO: Crawled 470 pages (at 1920 pages/min), scraped 0 items (at 0 items/min)
	2016-12-16 21:18:59 [scrapy.core.engine] INFO: Closing spider (closespider_timeout)
	2016-12-16 21:18:59 [scrapy.extensions.logstats] INFO: Crawled 518 pages (at 2880 pages/min), scraped 0 items (at 0 items/min)
	2016-12-16 21:19:00 [scrapy.statscollectors] INFO: Dumping Scrapy stats:
	{'downloader/request_bytes': 229995,
	 'downloader/request_count': 534,
	 'downloader/request_method_count/GET': 534,
	 'downloader/response_bytes': 1565504,
	 'downloader/response_count': 534,
	 'downloader/response_status_count/200': 534,
	 'finish_reason': 'closespider_timeout',
	 'finish_time': datetime.datetime(2016, 12, 16, 16, 19, 0, 647725),
	 'log_count/INFO': 17,
	 'request_depth_max': 19,
	 'response_received_count': 534,
	 'scheduler/dequeued': 533,
	 'scheduler/dequeued/memory': 533,
	 'scheduler/enqueued': 10661,
	 'scheduler/enqueued/memory': 10661,
	 'start_time': datetime.datetime(2016, 12, 16, 16, 18, 49, 799869)}
	2016-12-16 21:19:00 [scrapy.core.engine] INFO: Spider closed (closespider_timeout)

这说明了您的Scrapy能以3900页面/分钟的速度爬取。注意，这是一个非常简单，仅跟进链接的spider。 任何您所编写的spider会做更多处理，从而减慢爬取的速度。 减慢的程度取决于spider做的处理以及其是如何被编写的。

未来会有更多的用例会被加入到性能测试套装中，以覆盖更多常见的情景。