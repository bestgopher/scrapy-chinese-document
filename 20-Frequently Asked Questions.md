#常见问题(Frequently Asked Questions)#

</br>
##Scrapy相BeautifulSoup或lxml比较,如何呢(How does Scrapy compare to BeautifulSoup or lxml?)##

[BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/) 及 [lxml](https://lxml.de/) 是HTML和XML的分析库。Scrapy则是 编写爬虫，爬取网页并获取数据的应用框架(application framework)。

Scrapy提供了内置的机制来提取数据(叫做 选择器(selectors))。 但如果您觉得使用更为方便，也可以使用 BeautifulSoup (或 lxml)。 总之，它们仅仅是分析库，可以在任何Python代码中被导入及使用。

换句话说，拿Scrapy与 BeautifulSoup (或 lxml) 比较就好像是拿 jinja2 与 Django 相比。

</br>
##我能在Scrapy中使用BeautifulSoup吗？(Can I use Scrapy with BeautifulSoup?)##

当然，你可以的。正如上面提到的，BeautifulSoup能够被用来在Scrapy的回调函数中解析HTML响应。你仅需要把响应的文本传入 <font color=red>`BeautifulSoup`</font>对象中，然后解析你想要的任何数据。

这里是一个使用BeautifulSoup的示例，使用的是 <font color=red>`lxml`</font> 作为HTML解析器。

	from bs4 import BeautifulSoup
	import scrapy
	
	
	class ExampleSpider(scrapy.Spider):
	    name = "example"
	    allowed_domains = ["example.com"]
	    start_urls = (
	        'http://www.example.com/',
	    )
	
	    def parse(self, response):
	        # use lxml to get decent HTML parsing speed
	        soup = BeautifulSoup(response.text, 'lxml')
	        yield {
	            "url": response.url,
	            "title": soup.h1.string
	        }


<font color=orange>
NOTE:</br>
<font color=red>`BeautifulSoup`</font> 支持多种HTML/XML解析器， [BeautifulSoup’s official documentation](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#specifying-the-parser-to-use)查看哪些解析器可用。
</font>

</br>
##Scrapy支持那些Python版本？(What Python versions does Scrapy support?)##

Scrapy支持CPyhton(Python默认实现)和PyPy(从PyPy5.9开始)下的Python2.7和python3.4+。Python2.6在Scrapy0.20时被移除支持。Python3在Scrapy1.1开始支持。PyPy在Scrapy1.5开始支持，PyPy3在Scrapy1.5开始支持。

<font color=orange>
NOTE:</br>
对于在windows系统上的python3来说，推荐使用Anaconda/Miniconda。
</font>

</br>
##Scrapy是否从Django中”剽窃”呢？(Did Scrapy “steal” X from Django?)##

也许吧，不过我们不喜欢这个词。我们认为 Django 是一个很好的开源项目，同时也是 一个很好的参考对象，所以我们把其作为Scrapy的启发对象。

我们坚信，如果有些事情已经做得很好了，那就没必要再重复制造轮子。这个想法，作为 开源项目及免费软件的基石之一，不仅仅针对软件，也包括文档，过程，政策等等。 所以，与其自行解决每个问题，我们选择从其他已经很好地解决问题的项目中复制想法(copy idea) ，并把注意力放在真正需要解决的问题上。

如果Scrapy能启发其他的项目，我们将为此而自豪。欢迎来抄(steal)我们！

</br>

##Scrapy支持HTTP代理么？(Does Scrapy work with HTTP proxies?)##

是的。(从Scrapy 0.8开始)通过HTTP代理下载中间件对HTTP代理提供了支持。参考 `HttpProxyMiddleware`.

</br>
##(How can I scrape an item with attributes in different pages?)##

参考 Passing additional data to callback functions.

</br>

##Scrapy退出，ImportError: Nomodule named win32api(Scrapy crashes with: ImportError: No module named win32api)##

这是个Twisted bug ，您需要安装 pywin32 

</br>

##我要如何在spider里模拟用户登录呢?(How can I simulate a user login in my spider?)##

参考 使用FormRequest.from_response()方法模拟用户登录.
</br>

##Scrapy是以广度优先还是深度优先进行爬取的呢？(Does Scrapy crawl in breadth-first or depth-first order?)##

默认情况下，Scrapy使用 LIFO 队列来存储等待的请求。简单的说，就是 深度优先顺序 。深度优先对大多数情况下是更方便的。如果您想以 广度优先顺序 进行爬取，你可以设置以下的设定:

	DEPTH_PRIORITY = 1
	SCHEDULER_DISK_QUEUE = 'scrapy.squeues.PickleFifoDiskQueue'
	SCHEDULER_MEMORY_QUEUE = 'scrapy.squeues.FifoMemoryQueue'

</br>
##我的Scrapy爬虫有内存泄露，怎么办?(My Scrapy crawler has memory leaks. What can I do?)##

参考 调试内存溢出.

另外，Python自己也有内存泄露，在 Leaks without leaks 有所描述。
</br>
##如何让Scrapy减少内存消耗?(How can I make Scrapy consume less memory?)##

参考上一个问题

##我能在spider中使用基本HTTP认证么？(Can I use Basic HTTP Authentication in my spiders?)##

可以。参考 `HttpAuthMiddleware`.
</br>

##为什么Scrapy下载了英文的页面，而不是我的本国语言？(Why does Scrapy download pages in English instead of my native language?)##
尝试通过覆盖 `DEFAULT_REQUEST_HEADERS` 设置来修改默认的 Accept-Language 请求头。

</br>
##(Where can I find some example Scrapy projects?)##
参考 例子.

</br>

##我能在不创建Scrapy项目的情况下运行一个爬虫(spider)么？(Can I run a spider without creating a project?)##

是的。您可以使用 `runspider` 命令。例如，如果您有个 spider写在 <font color=red>`my_spider.py`</font> 文件中，您可以运行:

	scrapy runspider my_spider.py

详情请参考 `runspider` 命令。

</br>

##我收到了 “Filtered offsite request” 消息。如何修复？(I get “Filtered offsite request” messages. How can I fix them?)##

这些消息(以 <font color=red>`DEBUG`</font> 所记录)并不意味着有问题，所以你可以不修复它们。

这些消息由Offsite Spider中间件(Middleware)所抛出。 该(默认启用的)中间件筛选出了不属于当前spider的站点请求。

更多详情请参见: `OffsiteMiddleware`.
</br>

##发布Scrapy爬虫到生产环境的推荐方式？(What is the recommended way to deploy a Scrapy crawler in production?)##

参见 Scrapyd.

</br>

##我能对大数据(large exports)使用JSON么？(Can I use JSON for large exports?)##

这取决于您的输出有多大。参考 `JsonItemExporter` 文档中的 这个警告
</br>

##我能在信号处理器(signal handler)中返回(Twisted)引用么？(Can I return (Twisted) deferreds from signal handlers?)##

有些信号支持从处理器中返回引用，有些不行。参考 内置信号参考手册(Built-in signals reference) 来了解详情。
</br>

##reponse返回的状态值999代表了什么?(What does the response status code 999 means?)##

999是雅虎用来控制请求量所定义的返回值。 试着减慢爬取速度，将spider的下载延迟改为 2 或更高:

	class MySpider(CrawlSpider):
	
	    name = 'myspider'
	
	    download_delay = 2
	
	    # [ ... rest of the spider code ... ]

或在 `DOWNLOAD_DELAY` 中设置项目的全局下载延迟。
</br>

##我能在spider中调用 <font color=red>`pdb.set_trace()`</font> 来调试么？(Can I call <font color=red>`pdb.set_trace()`</font> from my spiders to debug them?)##

可以，但你也可以使用Scrapy终端。这能让你快速分析(甚至修改) spider处理返回的返回(response)。通常来说，比老旧的 <font color=red>`pdb.set_trace()`</font> 有用多了。

更多详情请参考 在spider中启动shell来查看response.</br>

##将所有爬取到的item转存(dump)到JSON/CSV/XML文件的最简单的方法?(Simplest way to dump all my scraped items into a JSON/CSV/XML file?)##

dump到JSON文件：

	scrapy crawl myspider -o items.json

dump到CSV文件：

	scrapy crawl myspider -o items.csv

dump到XML文件：

	scrapy crawl myspider -o items.xml
</br>

##在某些表单中巨大神秘的 <font color=red>`__VIEWSTATE`</font> 参数是什么？(What’s this huge cryptic <font color=red>`__VIEWSTATE`</font> parameter used in some forms?)##

<font color=red>`__VIEWSTATE`</font> 参数存在于ASP.NET/VB.NET建立的站点中。关于这个参数的作用请参考 [这篇文章](https://metacpan.org/pod/release/ECARROLL/HTML-TreeBuilderX-ASP_NET-0.09/lib/HTML/TreeBuilderX/ASP_NET.pm) 。这里有一个爬取这种站点的 样例爬虫 。

</br>

##分析大XML/CSV数据源的最好方法是?(What’s the best way to parse big XML/CSV data feeds?)##

使用XPath选择器来分析大数据源可能会有问题。选择器需要在内存中对数据建立完整的 DOM树，这过程速度很慢且消耗大量内存。

为了避免一次性读取整个数据源，您可以使用 <font color=red>`scrapy.utils.iterators`</font> 中的 <font color=red>`xmliter`</font> 及 <font color=red>`csviter`</font> 方法。 实际上，这也是feed spider(参考 Spiders)中的处理方法。\
</br>

##Scrapy自动管理cookies么？(Does Scrapy manage cookies automatically?)##

是的，Scrapy接收并保持服务器返回来的cookies，在之后的请求会发送回去，就像正常的网页浏览器做的那样。

更多详情请参考 Requests and Responses 及 CookiesMiddleware 。
</br>

##如何才能看到Scrapy发出及接收到的cookies呢？(How can I see the cookies being sent and received from Scrapy?)##

启用 `COOKIES_DEBUG` 选项。

</br>

##要怎么停止爬虫呢?(How can I instruct a spider to stop itself?)##

在回调函数中抛出 `CloseSpider` 异常。 更多详情请参见: CloseSpider 。

</br>

##如何避免我的Scrapy机器人(bot)被禁止(ban)呢？(How can I prevent my Scrapy bot from getting banned?)##

参考 避免被禁止(ban).
</br>

##我应该使用spider参数(arguments)还是设置(settings)来配置spider呢(Should I use spider arguments or settings to configure my spider?)##

spider参数 及 设置(settings) 都可以用来配置您的spider。 没有什么强制的规则来限定要使用哪个，但设置(settings)更适合那些一旦设置就不怎么会修改的参数， 而spider参数则意味着修改更为频繁，在每次spider运行都有修改，甚至是spider运行所必须的元素 (例如，设置spider的起始url)。

这里以例子来说明这个问题。假设您有一个spider需要登录某个网站来 爬取数据，并且仅仅想爬取特定网站的特定部分(每次都不一定相同)。 在这个情况下，认证的信息将写在设置中，而爬取的特定部分的url将是spider参数

</br>

##我爬取了一个XML文档但是XPath选择器不返回任何的item(I’m scraping a XML document and my XPath selector doesn’t return any items)##

也许您需要移除命名空间(namespace)。参见 移除命名空间.

