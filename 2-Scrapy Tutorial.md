# Scrapy教程(Scrapy Tutorial) #

在这个教程，我们将假定你的系统上面已经安装好了Scrapy。如果不是这种情况，参考安装指导。

我们将继续解剖 *quotes.toscrape.com*， 一个列出许多名人引用的网站。

这个教程将指导你一步一步完成以下任务：

1. 创建一个新的Scrapy项目
2. 写一个爬虫去爬取网站和提取数据
3. 用命令行导出已爬取的数据
4. 改变爬虫递归地爬取下一页
5. 使用爬虫参数

Scrapy是用python写的。如果你对这个语言不熟悉，你需要先更多地了解这门语言是怎样的，以便于充分使用Scrapy。

如果你已经熟悉其他语言了，想要快速学习pyhton，我们推荐阅读《Dive Into Python 3》。或者，你可以按照python教程操作。

如果你不熟悉编程，想从pyhton开始，你可以找到有用的在线书籍《Learn Python The Hard Way》。你也可以看看《this list of Python resources for non-programmers》。


## 创建一个项目(Creating a project) ##

在你开始爬取之前，你将必须建立一个新的Scrapy项目。进入你想要存储代码的目录，然后运行：

    scrapy startproject tutorial

这将创建一个 <font color=red>`tutorial`</font> 的目录，内容如下：

	tutorial/
	    scrapy.cfg            # deploy configuration file
	
	    tutorial/             # project's Python module, you'll import your code from here
	        __init__.py
	
	        items.py          # project items definition file
	
	        middlewares.py    # project middlewares file
	
	        pipelines.py      # project pipelines file
	
	        settings.py       # project settings file
	
	        spiders/          # a directory where you'll later put your spiders
	            __init__.py



## 我们的第一个爬虫(Our first spider) ##

爬虫是你定义的类，Scrapy使用它从一个网站或者一组网站中爬取信息。它们必须继承自 **scrapy.Spider**，必须定义生成初始的请求，怎么定位这个页面的链接和解析下载下来的页面内容来提取数据都是随意的。

这是我们第一个爬虫的代码。把它保存为一个文件，命名为 <font color=red>`quotes_spider.py`</font> ，放在你的项目中的   <font color=red>`tutorial/spiders`</font> 目录下。

	import scrapy
	
	
	class QuotesSpider(scrapy.Spider):
	    name = "quotes"
	
	    def start_requests(self):
	        urls = [
	            'http://quotes.toscrape.com/page/1/',
	            'http://quotes.toscrape.com/page/2/',
	        ]
	        for url in urls:
	            yield scrapy.Request(url=url, callback=self.parse)
	
	    def parse(self, response):
	        page = response.url.split("/")[-2]
	        filename = 'quotes-%s.html' % page
	        with open(filename, 'wb') as f:
	            f.write(response.body)
	        self.log('Saved file %s' % filename)

正如你所看到的，我们的爬虫的继承自 **scrapy.Spider**，而且定义了一些属性和方法：

  - `name`: 爬虫的标识。它在一个项目中必须唯一，也就是说，你不能为不同的爬虫设置相同的名字。
  - `start_requests()`: 必须返回一个可迭代的请求对象(Requests)(可以返回请求的列表或者写一个生成器函数)，爬虫将从这些请求中开始爬取。后续的请求将从这些初始的请求中相继生成。
  - `parse()`: 一个方法，它将被调用来处理每个请求下载获取的响应。参数 *reponse* 是一个 `TextResponse` 类的实例，保存了页面的文本内容，并且有一些好用的方法来处理它。
  	
    `parse()` 方法通常用来解析响应，提取爬取的数据生成字典，也寻找新的URLs来生成新的requests (`Request`).

## 怎么运行爬虫(How to run spider) ##

要让我们的爬虫工作，在项目顶层目录运行:

    scrapy crawl quotes

这个命令将运行我们添加的名为 <font color=red>`quotes`</font> 的爬虫，它将会向域名为 <font color=red>quotes.toscrape.com`</font> 的网站发送一些请求。你将获得一些类似于这些的输出：

	... (omitted for brevity)
	2016-12-16 21:24:05 [scrapy.core.engine] INFO: Spider opened
	2016-12-16 21:24:05 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
	2016-12-16 21:24:05 [scrapy.extensions.telnet] DEBUG: Telnet console listening on 127.0.0.1:6023
	2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (404) <GET http://quotes.toscrape.com/robots.txt> (referer: None)
	2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
	2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/2/> (referer: None)
	2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-1.html
	2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-2.html
	2016-12-16 21:24:05 [scrapy.core.engine] INFO: Closing spider (finished)
	...

现在，检查当前目录的文件。你应该会注意到两个新文件被创建: *quotes-.html* 和 *quotes-2.html*，它们中有各自url的对应的网页内容，像我们 <font color=red>`parse()`</font> 方法中所规定的一样

<font color=#0099ff>
NOTE:如果你很好奇我们为什么还没有解析页面，请继续，我们将很快介绍它。
</font>

## 在底层发生了什么(What just happend under the hood) ##

Scrapy调度从爬虫的 <font color=red>`start_requests`</font> 方法中返回的 `scrapy.Request` 请求对象。根据每个请求对象获得的响应，它会被 `Response` 实例化，然后把响应的实例化对象作为参数调用请求对象关联的回调方法(在此示例中，为 <font color=red>`parse()`</font> 方法)。

## start\_requests方法的快捷方式(A shortcut to the start_requests method) ##

你可以定义一个 `start_urls` 的类属性来绑定一个URLs的列表，来代替实现 `start_requests()` 方法通过URLs来生成 `scrapy.Request` 对象。这个列表接下会在默认实现的 `start_requests()` 方法中被使用，为你的爬虫创建初始的请求。

	import scrapy
	
	
	class QuotesSpider(scrapy.Spider):
	    name = "quotes"
	    start_urls = [
	        'http://quotes.toscrape.com/page/1/',
	        'http://quotes.toscrape.com/page/2/',
	    ]
	
	    def parse(self, response):
	        page = response.url.split("/")[-2]
	        filename = 'quotes-%s.html' % page
	        with open(filename, 'wb') as f:
	            f.write(response.body)

`parse()` 方法将会被调用来处理这URLs的响应，即使我们没有明确告知Scrapy要这样。这是因为 `parse()` 是Scrapy默认的回调方法，没有明确声明回调函数的请求获得响应时，将会调用它。

## 提取数据(Extracting data) ##

最好的方法学习怎么通过Scrapy提取数据就是通过 **Scrapy shell** 尝试使用选择器。运行：

    scrapy shell 'http://quotes.toscrape.com/page/1/'


<font color=#0099ff>
NOTE:</br>在命令行运行scrapy shell时url要使用引号包围，否则当你的url中包含参数(即， &字符)的时候将不起作用。</br>
在windows中，使用双引号:

    scrapy shell "http://quotes.toscrape.com/page/1/"
</font>

你将看到类似如下的内容：

	[ ... Scrapy log here ... ]
	2016-09-19 12:09:27 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
	[s] Available Scrapy objects:
	[s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
	[s]   crawler    <scrapy.crawler.Crawler object at 0x7fa91d888c90>
	[s]   item       {}
	[s]   request    <GET http://quotes.toscrape.com/page/1/>
	[s]   response   <200 http://quotes.toscrape.com/page/1/>
	[s]   settings   <scrapy.settings.Settings object at 0x7fa91d888c10>
	[s]   spider     <DefaultSpider 'default' at 0x7fa91c8af990>
	[s] Useful shortcuts:
	[s]   shelp()           Shell help (print this help)
	[s]   fetch(req_or_url) Fetch request (or URL) and update local objects
	[s]   view(response)    View response in a browser
	>>>

	
使用这个shell，你可以在 *response* 对象上通过 *CSS* 选择节点。

	>>> response.css('title')
	[<Selector xpath='descendant-or-self::title' data='<title>Quotes to Scrape</title>'>]

运行 <font color=red>`response.css('title')`</font> 获得的结果是一个类似于列表的对象，称为 `Selectorlist`，它表示一列 *Selector* 对象的集合，*Selector* 对象包含XML/HTML节点并允许你进一步细化选择和提取数据。

为了提取上面标题的文本内容，你可以这样：

	>>> response.css('title::text').extract()
	['Quotes to Scrape']

这里有两点需要注意：

一是我们要在CSS语句中增加 <font color=red>`::text`</font>，这意味着我们想从<font color=red>`<title>`</font>节点中选择文本内容。如果我们没有指定 <font color=red>`::text`</font>, 我们将得到整个节点，包括标签：
	
	>>> response.css('title').extract()
	['<title>Quotes to Scrape</title>']

二是调用 <font color=red>`.extract()`</font> 方法得到的结果是一个列表，因为我们正在处理一个 *Selectorlist* 实例。当你知道你仅仅想要第一个结果时，在这个示例中，你可以这样:

    >>> response.css('title::text').extract_first()
    'Quotes to Scrape'

你也可以这样写：
	
	>>> response.css('title::text')[0].extract()
	'Quotes to Scrape'

然而，当我们没有获得满足选择的节点，使用 <font color=red>`.extract_first()`</font> 避免 <font color=red>`IndexError`</font>，会返回 <font color=red>`None`</font>。

这里有个教训：对于大多数代码而言，你想要弹性规避因为没有找到正确的页面或者爬取页面失败而产生错误，使用<font color=red>`.extract_first()`</font>至少能得到返回值(None)。

除了 <font color=red>`extract()`</font> 和 <font color=red>`extract_first()`</font> 以外，你也可以通过使用 <font color=red>`re()`</font> 方法来使用正则表达式提取。

	>>> response.css('title::text').re(r'Quotes.*')
	['Quotes to Scrape']
	>>> response.css('title::text').re(r'Q\w+')
	['Quotes']
	>>> response.css('title::text').re(r'(\w+) to (\w+)')
	['Quotes', 'Scrape']

为了找到合适的CSS选择器使用，你可以在shell中使用 <font color=red>`view(response)`</font> 命令用你的浏览器打开当前响应的页面。这样你就可以使用浏览器的开发者工具或者 *Firebug* 这样的扩展工具了。

*Seletor Gadget* 是一个可以快速寻找CSS选择器的工具，在多种浏览器中都可以使用。

## XPath:简要介绍(XPath:a brief intro) ##

除了CSS， Scrapy选择器也支持使用XPath表达式。
	
	>>> response.xpath('//title')
	[<Selector xpath='//title' data='<title>Quotes to Scrape</title>'>]
	>>> response.xpath('//title/text()').extract_first()
	'Quotes to Scrape'

XPath表达式是非常强大的，是Scrapy选择的基础。事实上，CSS选择器在底层会被转化为XPath。
如果你仔细阅读选择器的文本表示形式，你将能看到。

或许不像CSS选择器那样受欢迎，但是XPath提供更多功能，除了提取结构数据之外，它还可以寻找文本内容。使用XPath，你可以提取：包含'Next page'这样文本的链接。这让XPath非常适合爬任务，即使你已经知道怎么构造一个CSS选择器， 我们还是鼓励你学习XPath，它会让爬虫变得更加简单。

我们在这里不会介绍太多关于XPath的内容，请自学。

## 提取quotes和authors(Extracting quotes and authors) ##

现在你已经了解了一些关于选择和提取的知识，让我们通过书写代码来完成我们的爬虫，提取从web页面中提取quotes。

在 *http://quotes.toscrape.com* 中，每个quote都是以以下HTML元素表现出来的：

	<div class="quote">
	    <span class="text">“The world as we have created it is a process of our
	    thinking. It cannot be changed without changing our thinking.”</span>
	    <span>
	        by <small class="author">Albert Einstein</small>
	        <a href="/author/Albert-Einstein">(about)</a>
	    </span>
	    <div class="tags">
	        Tags:
	        <a class="tag" href="/tag/change/page/1/">change</a>
	        <a class="tag" href="/tag/deep-thoughts/page/1/">deep-thoughts</a>
	        <a class="tag" href="/tag/thinking/page/1/">thinking</a>
	        <a class="tag" href="/tag/world/page/1/">world</a>
	    </div>
	</div>

让我们打开 Scrapy shell，演示以下怎么提取我们想要的数据：

    $ scrapy shell 'http://quotes.toscrape.com'

我们通过下面的代码获取一列quote的HTMl元素的Selector对象：

    >>> response.css("div.quote")

上面返回的每个选择器对象都允许我们进一步提取子元素。我们给第一个选择器对象声明一个变量，这样我们可以在特定的 *quote* 变量上直接运行CSS选择器。

    >>> quote = response.css("div.quote")[0]

现在，我们从我们创建的 <font color=red>`quote`</font> 对象中提取 <font color=red>`title`</font> , <font color=red>`author`</font> 和它的 <font color=red>`tags`</font> 。

	>>> title = quote.css("span.text::text").extract_first()
	>>> title
	'“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'
	>>> author = quote.css("small.author::text").extract_first()
	>>> author
	'Albert Einstein'

得到的tags是一列字符串，我们可以使用 <font color=red>`.extract()`</font> 方法来获得它们：
	
	>>> tags = quote.css("div.tags a.tag::text").extract()
	>>> tags
	['change', 'deep-thoughts', 'thinking', 'world']

解决了怎么提取其中一个，我们可以遍历所有的quotes节点，把得到的结果放在一个字典中。

	>>> for quote in response.css("div.quote"):
	...     text = quote.css("span.text::text").extract_first()
	...     author = quote.css("small.author::text").extract_first()
	...     tags = quote.css("div.tags a.tag::text").extract()
	...     print(dict(text=text, author=author, tags=tags))
	{'tags': ['change', 'deep-thoughts', 'thinking', 'world'], 'author': 'Albert Einstein', 'text': '“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'}
	{'tags': ['abilities', 'choices'], 'author': 'J.K. Rowling', 'text': '“It is our choices, Harry, that show what we truly are, far more than our abilities.”'}
	    ... a few more of these, omitted for brevity
	>>>

## 在我们的爬虫中提取数据(Extracting data in our spider ) ##

回到我们的爬虫。目前为止，没有提取任何数据，仅仅是在本地的文件中保存了整个HTML页面。我们将上面的提取逻辑集成到我们的爬虫中吧。

一个Scrapy爬虫通常会生成许多我们提取数据组成的字典。为此，我们在回调方法中使用python的关键字 <font color=red>`yield`</font> ，如下所示：

	import scrapy
	
	
	class QuotesSpider(scrapy.Spider):
	    name = "quotes"
	    start_urls = [
	        'http://quotes.toscrape.com/page/1/',
	        'http://quotes.toscrape.com/page/2/',
	    ]
	
	    def parse(self, response):
	        for quote in response.css('div.quote'):
	            yield {
	                'text': quote.css('span.text::text').extract_first(),
	                'author': quote.css('small.author::text').extract_first(),
	                'tags': quote.css('div.tags a.tag::text').extract(),
	            }

如果你运行这个爬虫，提取的数据将会在log中显示出来。

	2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
	{'tags': ['life', 'love'], 'author': 'André Gide', 'text': '“It is better to be hated for what you are than to be loved for what you are not.”'}
	2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
	{'tags': ['edison', 'failure', 'inspirational', 'paraphrased'], 'author': 'Thomas A. Edison', 'text': "“I have not failed. I've just found 10,000 ways that won't work.”"}


## 存储已爬取的内容(Storing the scraped data) ##

最简单存储数据的方法是使用 **Feed Exports**， 使用如下的命令：

    scrapy crawl quotes -o quotes.json


这将会生成一个包含所有爬取到的项目的 <font color=red>`quotes.json`</font> 文件，通过JSON序列化。 

由于历史原因，Scrapy会在给定的文件中追加内容，而不是覆盖原有的内容。如果你执行两次命令，并且在第一次命令之前没有移除这个文件，你将会得到一个损坏的JSON文件。

你可以使用其他格式，比如JSON Lines：

    scrapy crawl quotes -o quotes.jl


JOSN Lines 格式很有用的，因为它是流式的。你可以很简单的追加新的记录。当你执行两次命令的时候，不会出现和JSON格式一样的问题。而且每个记录都是单独的一行，你可以不用在内容中也能处理大文件，向JQ这样的工具能帮助我们在命令行操作这些事情。

在小项目(如我们教程中的项目)中，用文件存储就能满足需求了。然而，如果你想要使用以爬取的item做更加复杂的事情，我们可以写一个 **Item Pipeline**。当项目创建的时候，**Item Pipeline** 的占位文件就已经创建好了， 就是 <font color=red>`tutorial/pipelines.py`</font>。如果你仅仅只想要存储items，你不需要使用任何 **Item Pipelines**。

## 获取链接(Following links) ##

你不仅仅需要 http://quotes.toscrape.com的前两页内容，你想要从这个网站的所有页面获取quotes。

现在你知道怎么从页面获取数据，让我们看看怎么从页面获取链接。

第一件事就是从页面中提取我们想要的链接。以我们的页面举例，我们可以从以下标签中看到有一个去往下一页的链接。

	<ul class="pager">
	    <li class="next">
	        <a href="/page/2/">Next <span aria-hidden="true">&rarr;</span></a>
	    </li>
	</ul>


我们在shell中可以这样提取它：

	>>> response.css('li.next a').extract_first()
	'<a href="/page/2/">Next <span aria-hidden="true">→</span></a>'

这获得了锚节点，但是我们想要属性 <font color=red>`href`</font>。为此，Scrapy支持CSS扩展让我们能够得到属性的文本内容，例如：

	>>> response.css('li.next a::attr(href)').extract_first()
	'/page/2/'

修改我们的爬虫递归地获得下一页的链接，从中提取内容：

	import scrapy
	
	
	class QuotesSpider(scrapy.Spider):
	    name = "quotes"
	    start_urls = [
	        'http://quotes.toscrape.com/page/1/',
	    ]
	
	    def parse(self, response):
	        for quote in response.css('div.quote'):
	            yield {
	                'text': quote.css('span.text::text').extract_first(),
	                'author': quote.css('small.author::text').extract_first(),
	                'tags': quote.css('div.tags a.tag::text').extract(),
	            }
	
	        next_page = response.css('li.next a::attr(href)').extract_first()
	        if next_page is not None:
	            next_page = response.urljoin(next_page)
	            yield scrapy.Request(next_page, callback=self.parse)


现在，在提取完数据之后，<font color=red>`parse()`</font> 方法寻找下一页的链接，用  <font color=red>`urljoin()`</font>方法构建完整的绝对url(因为这里的url是相对的)，然后
yield 一个新的请求到下一页，把自己注册为下一页处理数据的回调方法，让爬虫能够遍历每一页。

这里你能看到Scrapy处理链接的机制：当你在一个回调方法中yield一个请求时，Scrapy将会调度发送请求，请求结束后，注册的回调方法将被执行。

使用这个，你可以通过你定义的规则提取的链接来构建完整的爬虫，根据访问的不同页面提取不同种类的数据。

在我们的示例中，它创建了一种循环，沿着下一页的链接爬取知道没有下一页为止-用于爬行博客、论坛和其他具有分页的网站。


## 创建请求对象的快捷方式(A shortcut for creating request) ##


你可以使用 <font color=red>`response.follow()`</font> 方法快捷创建request对象。

	import scrapy
	
	
	class QuotesSpider(scrapy.Spider):
	    name = "quotes"
	    start_urls = [
	        'http://quotes.toscrape.com/page/1/',
	    ]
	
	    def parse(self, response):
	        for quote in response.css('div.quote'):
	            yield {
	                'text': quote.css('span.text::text').extract_first(),
	                'author': quote.css('span small::text').extract_first(),
	                'tags': quote.css('div.tags a.tag::text').extract(),
	            }
	
	        next_page = response.css('li.next a::attr(href)').extract_first()
	        if next_page is not None:
	            yield response.follow(next_page, callback=self.parse)


与 <font color=red>`scrapy.Request`</font> 不同， <font color=red>`reponse.follow`</font> 直接支持相对url-不需要调用 <font color=red>`response.urljoin()`</font>。记住：<font color=red>`reponse.follow`</font> 仅仅是返回一个 **Request** 示例，我们仍然需要 *yield* 这个 **Request**。

你可以给 <font color=red>`reponse.follow`</font> 传递一个Selector对象，而不是一个字符串。这个 Selector 应该能提取有必要的属性：

	for href in response.css('li.next a::attr(href)'):
	    yield response.follow(href, callback=self.parse)

对于<font color=red>`<a>`</font>标签， <font color=red>`response.follow`</font> 自动地使用它的 *href* 属性。因此代码可以这样进一步缩短：

	for a in response.css('li.next a'):
	    yield response.follow(a, callback=self.parse)


<font color=#0099ff>
NOTE:</br> <font color=red>`response.follow(response.css('li.next a'))`</font> 不是有效的，因为 <font color=red>`reponse.css`</font> 返回一个所有结果的 Selector 组成的类似列表的对象， 不是单个的 Selector 。像本例中使用for循环或者 <font color=red>`response.css('li.next a')[0]`</font> 就行了。
</font>

## 更多示例和模式(More example and patterns) ##

这里是另一个爬虫，阐明回调和沿着url爬取，这次我们爬取的是作者信息。

	import scrapy
	
	
	class AuthorSpider(scrapy.Spider):
	    name = 'author'
	
	    start_urls = ['http://quotes.toscrape.com/']
	
	    def parse(self, response):
	        # follow links to author pages
	        for href in response.css('.author + a::attr(href)'):
	            yield response.follow(href, self.parse_author)
	
	        # follow pagination links
	        for href in response.css('li.next a::attr(href)'):
	            yield response.follow(href, self.parse)
	
	    def parse_author(self, response):
	        def extract_with_css(query):
	            return response.css(query).extract_first().strip()
	
	        yield {
	            'name': extract_with_css('h3.author-title::text'),
	            'birthdate': extract_with_css('.author-born-date::text'),
	            'bio': extract_with_css('.author-description::text'),
	        }


这个爬虫从主页面开始，它将沿着所有的链接爬取作者界面，然后调用 <font color=red>`parse_author`</font> 回调方法，当然分页链接仍然如我们之间所看见的一样用 <font color=red>`parse`</font> 回调。

这里我们把链接作为位置参数传递给 <font color=red>`reponse.follow`</font> 使得代码更短；当然也适用于 <font color=red>`scrapy.Request`</font>

<font color=red>`parse_author`</font> 回调方法定义了一个函数，这个函数通过CSS提取和清洗数据，然后 <font color=red>`parse_author`</font> 把作者数据组成的字典yied出来。

这个爬虫示例中另一件有趣的事情就是，即使这里有许多quotes出自相同的作者，我们不需要担心多次访问同一个作者的页面。默认的，Scrapy会筛选出已经访问过的URLs的重复请求，避免因代码问题而多次访问服务器的问题。这里可以通过设置来配置 **DUPEFILTER_CLASS**。

到目前为止希望你能很好的理解如何通过Scrapy跟踪链接和回调。

另一个实现跟踪链接机制的爬虫示例，请查看作为通用函数的 CrawlSpider 类，它实现了一个小型的规则引擎，你可以在这个规则引擎上编写你的爬虫。

另外，一个常见的模式就是创建一个来自许多页面数据的item，在回调的时候把数据加进item里面。

## 使用爬虫参数(Using spider arguments) ##

当你运行爬虫的时候，你可以在命令行的时候通过 <font color=red>`-a`</font> 选项为你的爬虫提供参数。
    
    scrapy crawl quotes -o quotes-humor.json -a tag=humor

这些参数将会默认传递给 **Spider** 的 <font color=red>`__init__`</font> 方法，成为爬虫的属性。

在这个示例中， <font color=red>`tag`</font> 的值可以在爬虫中用 <font color=red>`self.tag`</font> 使用。你可以使用它获取只有特殊 tag 的quotes，URL以tag构建而成。

	import scrapy
	
	
	class QuotesSpider(scrapy.Spider):
	    name = "quotes"
	
	    def start_requests(self):
	        url = 'http://quotes.toscrape.com/'
	        tag = getattr(self, 'tag', None)
	        if tag is not None:
	            url = url + 'tag/' + tag
	        yield scrapy.Request(url, self.parse)
	
	    def parse(self, response):
	        for quote in response.css('div.quote'):
	            yield {
	                'text': quote.css('span.text::text').extract_first(),
	                'author': quote.css('small.author::text').extract_first(),
	            }
	
	        next_page = response.css('li.next a::attr(href)').extract_first()
	        if next_page is not None:
	            yield response.follow(next_page, self.parse)

如果这个爬虫中你添加了 <font color=red>`tag=humor`</font>，你会意识到只会爬取 <font color=red>`humer`</font> 标签的URLs， 例如：<font color=red>`http://quotes.toscrape.com/tag/humor`</font>。
