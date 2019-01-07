# 初窥Scrapy(Scrapy at a glance) #
scrapy是一个爬取web网站和提取结构化数据的应用框架，作为一个有用的应用，被广泛应用于数据采集，信息处理和历史存档。

虽然scrapy最初设计为web爬取，它也可以用来通过使用APIs来提取数据(比如 Amazon Associates Web Services)或者作为通用网络爬虫来爬取数据。

## 一步步演示一个示例爬虫(Walk-through of an example spider) ##

为了向你展示Scrapy带来的内容，我们将使用最简单的方式运行一个爬虫以一步一步向你展示一个示例Scrapy爬虫。

以下是一个爬虫的代码，它通过页数爬取 *http://quotes.toscrape.com* 的引用。

	import scrapy
	
	
	class QuotesSpider(scrapy.Spider):
	    name = "quotes"
	    start_urls = [
	        'http://quotes.toscrape.com/tag/humor/',
	    ]
	
	    def parse(self, response):
	        for quote in response.css('div.quote'):
	            yield {
	                'text': quote.css('span.text::text').extract_first(),
	                'author': quote.xpath('span/small/text()').extract_first(),
	            }
	
	        next_page = response.css('li.next a::attr("href")').extract_first()
	        if next_page is not None:
	            yield response.follow(next_page, self.parse)



把这个复制到一个文件中，把它命名为类似 *quote_spider.py*，然后用 **runspider** 命令运行这个爬虫。
	
	scrapy runspider quotes_spider.py -o quotes.json

当这个爬虫结束后你将会在 *quotes.json* 文件中，以json格式得到引用(quotes)的清单，包含text和author，如下所示。

    [{
    "author": "Jane Austen",
    "text": "\u201cThe person, be it gentleman or lady, who has not pleasure in a good novel, must be intolerably stupid.\u201d"
	},
	{
	    "author": "Groucho Marx",
	    "text": "\u201cOutside of a dog, a book is man's best friend. Inside of a dog it's too dark to read.\u201d"
	},
	{
	    "author": "Steve Martin",
	    "text": "\u201cA day without sunshine is like, you know, night.\u201d"
	},
	...]


## 刚刚发生了什么(What just happened) ##

当你运行 **scrapy runspider quotes_spider.py** ，Scrapy会自动寻找这个爬虫然后通过Scrapy的爬虫引擎运行它。

爬虫开始于通过定义在**start_urls**(在这个示例中，只有humur类别的url)中的*URLs*生成请求，然后通过请求对象(response object)作为参数调用默认的回调方法(callback method) **parse**。在**parse**回调函数中，我们循环通过**CSS**选择器(CSS Selector)循环遍历*quote*元素，*yield*一个提取出来的text和author组成的Python字典，找到下一页的链接，使用相同的回调方法parse调度另一个请求。

这里你注意到Scrapy的优势之一：请求的调度和运行是异步的(requests are scheduled and processed asynchronously)。这意味着Scrapy不需要等待一个请求的运行和结束，它同时能发送另一个请求或者做其它事情。它也意味着当爬虫运行时即使一些请求失败或者发生错误其他的请求也能保持正常运行。

然而这能使你快速的爬取(在一个可容错的方式中同一时间发送多个并发请求)，但是Scrapy也可以通过一些设置让你能够控制爬虫在礼貌地运行。你可以做例如每个请求的下载延迟，控制请求每个域名或者没个ip并发的数量，甚至可以使用一些扩展(auto-throttling extension)去自动的尝试解决问题。

<font color=#0099ff>NOTE:这个示例使用了 **feed exports** 生成 *JSON* 文件，你可以很简单的改变导出格式(例如XML,CSV)或者存储后端(例如FTP或者Amazon S3)。你也可以写一个 **item pipeline** 来存储item到数据库中。</font>

## 还有什么？(What else?) ##

你已经了解了怎么使用Scrapy从一个网站提取和存储items，但是这仅仅是一个表面而已。Scrapy提供了许多功能强大的组件让爬虫更加容易和高效，例如：

- 内置支持使用扩展的CSS选择器和XPATH表达式从HTML/XML源码中选择和提取数据，并可以使用正则表达式辅助提取。
- 交互式的shell控制台(interactive shell console)(ipython形式的)。当我们在书写代码或者调试我们的爬虫的时候，可以使用控制台通过CSS或者XPATH表达式去试着匹配数据，这将非常有用。
- 内置支持生成feed exports。多种格式(JSON, CSV, XML)和多种后端(FTP, S3和本地文件系统)。
- 针对非英语语系中不标准或者错误的编码声明, 提供了自动检测以及健壮的编码支持。(broken encoding declarations)
- 支持强大的可扩展性，允许你使用信号和良好定义的API(中间件，扩展和管道)去插入你自己的功能。
- 大量的内置扩展和中间件来处理：
	- cookies和session、
	- HTTP功能，如压缩、身份验证、缓存
	- User-Agent伪装
	- robots.txt
	- 爬取深度限制
	- 等等
- 内置 Telnet终端 ，通过在Scrapy进程中钩入Python终端，使您可以查看并且调试爬虫
- 还有其他优点，例如可重复爬虫来从Sitemaps和XML/CSV导出中爬取，关联了爬取项目会使用媒体管道自动下载图片(和其他媒体文件)，具有缓存的DNS解析器等等。
