#通用实践(Common Practices)#

本节介绍使用Scrapy时的常见做法。这些内容涉及许多主题，并且通常不属于任何其他特定部分。

##在脚本中运行Scrapy(Run Scrapy from a script)##

除了常用的 <font color=red>`scrapy crawl`</font> 来启动Scrapy，您也可以使用 API 在脚本中启动Scrapy。

需要注意的是，Scrapy是在Twisted异步网络库上构建的， 因此其必须在Twisted reactor里运行。

第一个你可以用来运行你的spider的实用程序是 `scrapy.crawler.CralwerProcess`。这个类将会为你开启一个Twisted reactor，配置日志记录和设置关闭处理程序。这个类是所有命令使用的类。

这是一个演示怎么通过它运行一个简单spider的例子：

	import scrapy
	from scrapy.crawler import CrawlerProcess
	
	class MySpider(scrapy.Spider):
	    # Your spider definition
	    ...
	
	process = CrawlerProcess({
	    'USER_AGENT': 'Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)'
	})
	
	process.crawl(MySpider)
	process.start() # the script will block here until the crawling is finished

请务必查看`CrawlerProcess`文档，了解它的使用细节。

如果你在一个项目之内，这里有一些额外的功能供你使用，你需要在你的项目中导入这些组件。你可以通过spider的名字自动导入到`CrawlerProcess`，使用 <font color=red>`get_project_settings`</font> 获取你的项目设置的 `Settings` 实例。

下面是使用[testspider](https://github.com/scrapinghub/testspiders)项目为示例，演示如何操作。

	from scrapy.crawler import CrawlerProcess
	from scrapy.utils.project import get_project_settings
	
	process = CrawlerProcess(get_project_settings())
	
	# 'followall' is the name of one of the spiders of the project.
	process.crawl('followall', domain='scrapinghub.com')
	process.start() # the script will block here until the crawling is finished

有另一个Scrapy实用程序，它为爬取进程提供了更多的控制：`scrapy.crawler.CrawlerRunner`。这个类是一个精简的包装，封装了一些简单的helper来运行多个爬虫，但是不会以任何方式开启或者干扰现存在的reactors。

使用这个类，在调度你的spider后明确运行reactor。如果你的应用程序已经使用了Twisted，你希望使用相同的reactor运行你的Scrapy，推荐你使用`CrawlerRunner`代替`CrawlerProcess`。

记住在spider结束之后你必须自己关闭Twisted reactor。这个可以通过向`CrawlerRunner.crawl`方法的延迟返回添加回调函数来实现。

这是一个它的使用示例，在*Myspider*停止运行后通过回调函数手动的停止reactor。

	from twisted.internet import reactor
	import scrapy
	from scrapy.crawler import CrawlerRunner
	from scrapy.utils.log import configure_logging
	
	class MySpider(scrapy.Spider):
	    # Your spider definition
	    ...
	
	configure_logging({'LOG_FORMAT': '%(levelname)s: %(message)s'})
	runner = CrawlerRunner()
	
	d = runner.crawl(MySpider)
	d.addBoth(lambda _: reactor.stop())
	reactor.run() # the script will block here until the crawling is finished

##在同一进程中运行多个spider(Running multiple spiders in the same process)##

默认情况下，当你运行<font color=red>`scrapy crawl`</font>命令时，Scrapy中一个进程运行一个单独的spider。然而，通过使用内部API，Scrapy支持一个进程运行多个spider。

这是一个同时运行多个spiders的示例：

	import scrapy
	from scrapy.crawler import CrawlerProcess
	
	class MySpider1(scrapy.Spider):
	    # Your first spider definition
	    ...
	
	class MySpider2(scrapy.Spider):
	    # Your second spider definition
	    ...
	
	process = CrawlerProcess()
	process.crawl(MySpider1)
	process.crawl(MySpider2)
	process.start() # the script will block here until all crawling jobs are finished

使用 `CrawlerRunner`的相同示例：

	import scrapy
	from twisted.internet import reactor
	from scrapy.crawler import CrawlerRunner
	from scrapy.utils.log import configure_logging
	
	class MySpider1(scrapy.Spider):
	    # Your first spider definition
	    ...
	
	class MySpider2(scrapy.Spider):
	    # Your second spider definition
	    ...
	
	configure_logging()
	runner = CrawlerRunner()
	runner.crawl(MySpider1)
	runner.crawl(MySpider2)
	d = runner.join()
	d.addBoth(lambda _: reactor.stop())
	
	reactor.run() # the script will block here until all crawling jobs are finished

相同的示例，但是是通过链接的延迟循序地运行spiders：

	from twisted.internet import reactor, defer
	from scrapy.crawler import CrawlerRunner
	from scrapy.utils.log import configure_logging
	
	class MySpider1(scrapy.Spider):
	    # Your first spider definition
	    ...
	
	class MySpider2(scrapy.Spider):
	    # Your second spider definition
	    ...
	
	configure_logging()
	runner = CrawlerRunner()
	
	@defer.inlineCallbacks
	def crawl():
	    yield runner.crawl(MySpider1)
	    yield runner.crawl(MySpider2)
	    reactor.stop()
	
	crawl()
	reactor.run() # the script will block here until the last crawl call is finished



##分布式爬取(Distributed crawls)##

Scrapy并没有提供内置的机制支持分布式(多服务器)爬取。不过还是有办法进行分布式爬取， 取决于您要怎么分布了。

如果您有很多spider，那分布负载最简单的办法就是启动多个Scrapyd，并分配到不同机器上。

如果想要在多个机器上运行一个单独的spider，那您可以将要爬取的url进行分块，并发送给spider。 例如:

首先，准备要爬取的url列表，并分配到不同的文件/url里:
	
	http://somedomain.com/urls-to-crawl/spider1/part1.list
	http://somedomain.com/urls-to-crawl/spider1/part2.list
	http://somedomain.com/urls-to-crawl/spider1/part3.list

接着在3个不同的Scrapd服务器中启动spider。spider会接收一个(spider)参数 <font color=red>`part`</font> ， 该参数表示要爬取的分块:

	curl http://scrapy1.mycompany.com:6800/schedule.json -d project=myproject -d spider=spider1 -d part=1
	curl http://scrapy2.mycompany.com:6800/schedule.json -d project=myproject -d spider=spider1 -d part=2
	curl http://scrapy3.mycompany.com:6800/schedule.json -d project=myproject -d spider=spider1 -d part=3


##避免被禁止(ban)(Avoiding getting banned)##

有些网站实现了特定的机制，以一定规则来避免被爬虫爬取。 与这些规则打交道并不容易，需要技巧，有时候也需要些特别的基础。 如果有疑问请考虑联系 商业支持 。

下面是些处理这些站点的建议(tips):

  - 使用user agent池，轮流选择之一来作为user agent。池中包含常见的浏览器的user agent(google一下一大堆)
  - 禁止cookies(参考 `COOKIES_ENABLED`)，有些站点会使用cookies来发现爬虫的轨迹。
  - 设置下载延迟(2或更高)。参考 `DOWNLOAD_DELAY` 设置。
  - 如果可行，使用 Google cache 来爬取数据，而不是直接访问站点。
  - 使用IP池。例如免费的 Tor项目 或付费服务(ProxyMesh)。一个开源的实现是 scraproxy，这是一个超级代理，你可以在这上面附加你自己的代理。
  - 使用高度分布式的下载器(downloader)来绕过禁止(ban)，您就只需要专注分析处理页面。这样的例子有: Crawlera

如果您仍然无法避免被ban，考虑联系 商业支持。