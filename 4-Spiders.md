# Spiders #

Spider类定义了如何爬取某个(或某些)网站。包括了爬取的动作(例如:是否跟进链接)以及如何从网页的内容中提取结构化数据(爬取item)。 换句话说，Spider就是您定义爬取的动作及分析某个网页(或者是有些网页)的地方。

对spider来说，爬取的循环类似下文:

  1. 以初始的URL初始化Request，并设置回调函数。 当该request下载完毕并返回时，将生成response，并作为参数传给该回调函数。spider中初始的request是通过调用 `start_requests()` 来获取的。 `start_requests()` 读取 `start_urls` 中的URL， 并以 `parse` 为回调函数生成 Request。
  2. 在回调函数中，你解析响应(网页),返回带有提取数据的字典，`Item`对象，`Request`对象，或者这些对象的可迭代对象。这些请求也要包含一个回调函数(可能相同)，然后经过Scrapy下载相应的内容，并调用设置的callback函数(函数可相同)处理。
  3. 在回调函数内，您可以使用 选择器(Selectors) (您也可以使用BeautifulSoup, lxml 或者您想用的任何解析器) 来分析网页内容，并根据分析的数据生成item。
  4. 最后，由spider返回的item将被存到数据库(由某些 Item Pipeline 处理)或使用 Feed exports 存入到文件中。

虽然该循环对任何类型的spider都(多少)适用，但Scrapy仍然为了不同的需求提供了多种默认spider。 之后将讨论这些spider。

## scrapy.Spider ##

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spiders.Spider
</td></tr></table>

Spider是最简单的spider。每个其他的spider必须继承自该类(包括Scrapy自带的其他spider以及您自己编写的spider)。 这个类并没有提供什么特殊的功能。它仅仅提供一个默认的 `start_requests()` 方法，这个方法从 `start_urls` 类属性中获取url，并发送请求，然后为每个结果响应调用`parse()` 方法。

**name**

定义spider名字的字符串(string)。spider的名字定义了Scrapy如何定位(并初始化)spider，所以其必须是唯一的。 不过您可以生成多个相同的spider实例(instance)，这没有任何限制。 name是spider最重要的属性，而且是必须的。

如果该spider爬取单个网站(single domain)，一个常见的做法是以该网站(domain)(加或不加 后缀 )来命名spider。 例如，如果spider爬取 <font color=red>`mywebsite.com`</font> ，该spider通常会被命名为 <font color=red>`mywebsite`</font> 。

<font color=orange>
NOTE：</br>
python2中必须为ASCII。
</font>

**allowed_domains**

可选。包含了spider允许爬取的域名(domain)列表(list)。 当 `OffsiteMiddleware` 启用时， 域名(或者子域名)不在列表中的URL不会被跟进。

假设您的目标网址是 <font color=red>`https://www.example.com/1.html`</font>，然后添加 <font color=red>`'example.com'`</font> 到列表中。

**start_urls**

URL列表。当没有制定特定的URL时，spider将从该列表中开始进行爬取。 因此，第一个被获取到的页面的URL将是该列表中的url之一。 后续的 `Request` 将会从起始url获取到的数据中提取。

**custom_settings**

一个settings的字典，运行spider时将会覆盖项目的配置。这个必须定义为类属性，因为settings在spider初始化之前更新。

**crawler**

这个属性是在 `from_crawler()` 类方法中，实例化spider后设置的，链接spider绑定的 `Crawler` 对象。

Crawler封装了许多项目中的组件(扩展，中间件，信号管理等)。

**settings**

运行这个Spider的配置选项。这是一个 `Settings` 实例。

**logger**

使用spider的 `name` 创建python日志记录器。你可以使用这个发送日志信息。

**from_crawler(crawler, \*args, \*\*kwargs)**

Scrapy使用这个类方法创建spider实例。

你或许不需要直接重写它，因为它默认实现了充当 `__init__()` 方法的代理。

尽管如此，这个方法为新的spider实例设置了 `crawler` 和 `settings` 属性，因此他们能在spider代码中访问这些属性。

&nbsp;&nbsp;&nbsp;**参数**：

  - **crawler**(`Crawler` 实例) - 这个spider要绑定的crawler
  - **args**(list) - 要传递给 `__init__()` 方法的参数
  - **kwargs**(dict) - 要传递给 `__init__()` 方法的关键字参数

**start_requests()**

该方法必须返回一个可迭代对象(iterable)。该对象包含了spider用于爬取的第一个Request。在spider开启爬取时被Scrapy调用。该方法仅仅会被Scrapy调用一次，因此您可以将其实现为生成器。

默认实现为`start_urls`中的url生成 <font color=red>`Request(url, dont_filter=True)`</font> 对象。

如果你想改变开始爬取的Requests，重写这个方法。例如，如果你需要一个POST请求登录，你可以这样：

	class MySpider(scrapy.Spider):
	    name = 'myspider'
	
	    def start_requests(self):
	        return [scrapy.FormRequest("http://www.example.com/login",
	                                   formdata={'user': 'john', 'pass': 'secret'},
	                                   callback=self.logged_in)]
	
	    def logged_in(self, response):
	        # here you would extract links to follow and return Requests for
	        # each of them, with another callback
	        pass

**parse(response)**

当response没有指定回调函数时，该方法是Scrapy处理下载的response的默认方法。

<font color=red>`parse`</font> 负责处理response并返回处理的数据以及(或)跟进的URL。 `Spider` 对其他的Request的回调函数也有相同的要求。

该方法及其他的Request回调函数必须返回一个包含 Request 及(或) 字典或Item 的可迭代的对象。

&nbsp;&nbsp;&nbsp;**参数**：

  - **response**(`Response` 对象) - 要解析的响应
  
**log(message[, level, component])**

封装spider的 `logger`，用于发送消息，保持后面版本的兼容。

**closed(reason)**

当spider关闭时，该函数被调用。 该方法提供了一个替代调用signals.connect()来监听 `spider_closed` 信号的快捷方式。 

让我们看个例子：

	import scrapy
	
	
	class MySpider(scrapy.Spider):
	    name = 'example.com'
	    allowed_domains = ['example.com']
	    start_urls = [
	        'http://www.example.com/1.html',
	        'http://www.example.com/2.html',
	        'http://www.example.com/3.html',
	    ]
	
	    def parse(self, response):
	        self.logger.info('A response from %s just arrived!', response.url) 

从同一个回调函数中返回多个Requests和items：

	import scrapy
	
	class MySpider(scrapy.Spider):
	    name = 'example.com'
	    allowed_domains = ['example.com']
	    start_urls = [
	        'http://www.example.com/1.html',
	        'http://www.example.com/2.html',
	        'http://www.example.com/3.html',
	    ]
	
	    def parse(self, response):
	        for h3 in response.xpath('//h3').extract():
	            yield {"title": h3}
	
	        for url in response.xpath('//a/@href').extract():
	            yield scrapy.Request(url, callback=self.parse)

除了 `start_urls` 你可以直接使用 `start_requests`；为数据提供更多结构，你可以使用Items：

	import scrapy
	from myproject.items import MyItem
	
	class MySpider(scrapy.Spider):
	    name = 'example.com'
	    allowed_domains = ['example.com']
	
	    def start_requests(self):
	        yield scrapy.Request('http://www.example.com/1.html', self.parse)
	        yield scrapy.Request('http://www.example.com/2.html', self.parse)
	        yield scrapy.Request('http://www.example.com/3.html', self.parse)
	
	    def parse(self, response):
	        for h3 in response.xpath('//h3').extract():
	            yield MyItem(title=h3)
	
	        for url in response.xpath('//a/@href').extract():
	            yield scrapy.Request(url, callback=self.parse)

## Spider参数(Spider arguments) ##

Spider可以通过接受参数来修改其功能。 spider参数一般用来定义初始URL或者指定限制爬取网站的部分。 您也可以使用其来配置spider的任何功能。

在运行 `crawl` 时添加 <font color=red>`-a`</font> 可以传递Spider参数:

scrapy crawl myspider -a category=electronics

spider在 `__init__` 方法中接受这些参数：

	import scrapy
	
	class MySpider(scrapy.Spider):
	    name = 'myspider'
	
	    def __init__(self, category=None, *args, **kwargs):
	        super(MySpider, self).__init__(*args, **kwargs)
	        self.start_urls = ['http://www.example.com/categories/%s' % category]
	        # ...

默认情况下 `__init__` 方法会获取任何spider参数，复制这些参数最为spider的属性。上面的例子也可以这样写：

	import scrapy
	
	class MySpider(scrapy.Spider):
	    name = 'myspider'
	
	    def start_requests(self):
	        yield scrapy.Request('http://www.example.com/categories/%s' % self.category)

记住spider参数只能为字符串。spider本身不会进行任何解析。如果你在命令行设置了 `start_urls` 属性，你必须使用例如 ast.literal\_eval 或者 json.loads 解析它为列表，然后设置它为一个属性。否则，你将造成对start_urls字符串进行迭代的问题(一个很常见的python陷阱)，得到的每一个字符串都被视为一个单独的url。

一个有效的使用情况是通过`HttpAuthMiddleware`设置http认证，或者通过`UserAgentMiddleware` 使用User-Agent。

	scrapy crawl myspider -a http_user=myuser -a http_pass=mypassword -a user_agent=mybot

Spider参数也可以通过Scrapyd的 <font color=red>`schedule.json`</font> API来传递。

## 通用spiders(Generic Spiders) ##

Scrapy附带了一些有用的通用spiders，你的spider可以继承他们。他们的目的是为一个共同的爬取场景提供便利的方法，例如，跟踪一个网站中满足某一规则的所有链接，从网站导航中爬取，或者解析为XML/CSV feed。

下面的例子中，我们假设有一个 <font color=red>`TestItem`</font>
 Item类，声明在 <font color=red>`myproject.items`</font> 模块中。
	
	import scrapy
	
	class TestItem(scrapy.Item):
	    id = scrapy.Field()
	    name = scrapy.Field()
	    description = scrapy.Field()

## CrawlSpider ##

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spiders.CrawlSpider
</td></tr></table>

爬取一般网站常用的spider。其定义了一些规则(rule)来提供跟进link的方便的机制。 也许该spider并不是完全适合您的特定网站或项目，但其对很多情况都使用。 因此您可以以其为起点，根据需求修改部分方法。当然您也可以实现自己的spider。

除了从Spider继承过来的(您必须提供的)属性外，其提供了一个新的属性:

**rules**

一个包含一个(或多个) `Rule` 对象的集合(list)。 每个 `Rule` 对爬取网站的动作定义了特定表现。 Rule对象在下边会介绍。 如果多个rule匹配了相同的链接，则根据他们在本属性中被定义的顺序，第一个会被使用。

该spider也提供了一个可复写(overrideable)的方法：

**parse\_start_url(response)**

当start_urls的请求返回时，该方法被调用。 该方法分析最初的返回值并必须返回一个 Item 对象或者 一个 Request 对象或 一个包含二者的可迭代对象。

### Crawling rules ###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spiders.Rule(link_extractor, callback=None, cb_kwargs=None, follow=None, process_links=None, process_request=None)
</td></tr></table>

<font color=red>`link_extractor`</font> 是一个 Link Extractor 对象。 其定义了如何从爬取到的页面提取链接。

<font color=red>`callback`</font> 是一个callable或string(该spider中同名的函数将会被调用)。 从link_extractor中每获取到链接时将会调用该函数。该回调函数接受一个response作为其第一个参数， 并返回一个包含 Item 以及(或) Request 对象(或者这两者的子类)的列表(list)。

<font color=orange></br>

当编写爬虫规则时，请避免使用 <font color=red>`parse`</font> 作为回调函数。 由于 `CrawlSpider` 使用 <font color=red>`parse`</font>  方法来实现其逻辑，如果 您覆盖了 <font color=red>`parse `</font> 方法，crawl spider 将会运行失败。

</font>

<font color=red>`cb_kwargs`</font> 包含传递给回调函数的参数(keyword argument)的字典。

<font color=red>`follow`</font> 是一个布尔(boolean)值，指定了根据该规则从response提取的链接是否需要跟进。 如果 <font color=red>`callback`</font> 为None， <font color=red>`follow`</font> 默认设置为 <font color=red>`True`</font> ，否则默认为 <font color=red>`False`</font> 。

<font color=red>`process_links`</font> 是一个callable或string(该spider中同名的函数将会被调用)。 从<font color=red>`link_extractor`</font>中获取到链接列表时将会调用该函数。该方法主要用来过滤。

<font color=red>`process_request`</font> 是一个callable或string(该spider中同名的函数将会被调用)。 该规则提取到每个request时都会调用该函数。该函数必须返回一个request或者None。 (用来过滤request)

### CrawlSpider样例(CrawlSpider example) ###

接下来给出配合rule使用CrawlSpider的例子：

	import scrapy
	from scrapy.spiders import CrawlSpider, Rule
	from scrapy.linkextractors import LinkExtractor
	
	class MySpider(CrawlSpider):
	    name = 'example.com'
	    allowed_domains = ['example.com']
	    start_urls = ['http://www.example.com']
	
	    rules = (
	        # Extract links matching 'category.php' (but not matching 'subsection.php')
	        # and follow links from them (since no callback means follow=True by default).
	        Rule(LinkExtractor(allow=('category\.php', ), deny=('subsection\.php', ))),
	
	        # Extract links matching 'item.php' and parse them with the spider's method parse_item
	        Rule(LinkExtractor(allow=('item\.php', )), callback='parse_item'),
	    )
	
	    def parse_item(self, response):
	        self.logger.info('Hi, this is an item page! %s', response.url)
	        item = scrapy.Item()
	        item['id'] = response.xpath('//td[@id="item_id"]/text()').re(r'ID: (\d+)')
	        item['name'] = response.xpath('//td[@id="item_name"]/text()').extract()
	        item['description'] = response.xpath('//td[@id="item_description"]/text()').extract()
	        return item

该spider将从example.com的首页开始爬取，获取category以及item的链接并对后者使用 <font color=red>`parse_item`</font> 方法。 当item获得返回(response)时，将使用XPath处理HTML并生成一些数据填入 `Item` 中。

## XMLFeedSpider ##

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spiders.XMLFeedSpider
</td></tr></table>

XMLFeedSpider被设计用于通过迭代各个节点来分析XML源(XML feed)。 迭代器可以从 <font color=red>`iternodes`</font> ， <font color=red>`xml`</font> ， <font color=red>`html`</font>  选择。 鉴于 <font color=red>`xml`</font> 以及 <font color=red>`html`</font> 迭代器需要先读取所有DOM再分析而引起的性能问题， 一般还是推荐使用 <font color=red>`iternodes`</font>  。 不过使用 <font color=red>`html`</font>  作为迭代器能有效应对错误的XML。

您必须定义下列类属性来设置迭代器以及标签名(tag name):

**iterator**

用于确定使用哪个迭代器的string。可选项有：

  - <font color=red>`'iternodes'`</font> - 一个高性能的基于正则表达式的迭代器
  - <font color=red>`'html'`</font> - 使用 Selector 的迭代器。 需要注意的是该迭代器使用DOM进行分析，其需要将所有的DOM载入内存， 当数据量大的时候会产生问题。
  - <font color=red>`'xml'`</font> - 使用 Selector 的迭代器。 需要注意的是该迭代器使用DOM进行分析，其需要将所有的DOM载入内存， 当数据量大的时候会产生问题。

默认值为 <font color=red>`iternodes`</font> 。

**itertag**

一个包含开始迭代的节点名的string。例如：

	itertag = 'product'

**namespaces**

一个由 <font color=red>`(prefix, uri)`</font> 元组(tuple)所组成的list。 其定义了在该文档中会被spider处理的可用的namespace。 <font color=red>`refix`</font> 及 <font color=red>`uri`</font> 会被自动调用 register_namespace() 生成namespace。

您可以通过在 itertag 属性中制定节点的namespace。

例如：

	class YourSpider(XMLFeedSpider):
	
	    namespaces = [('n', 'http://www.sitemaps.org/schemas/sitemap/0.9')]
	    itertag = 'n:url'
	    # ...

除了这些新的属性之外，该spider也有以下可以覆盖(overrideable)的方法:

**adapt_response(response)**

该方法在spider分析response前被调用。您可以在response被分析之前使用该函数来修改内容(body)。 该方法接受一个response并返回一个response(可以相同也可以不同)。

**parse_node(response, selector)**

当节点符合提供的标签名时(<font color=red>`itertag`</font>)该方法被调用。 接收到的response以及相应的 `Selector` 作为参数传递给该方法。 该方法返回一个 Item 对象或者 Request 对象 或者一个包含二者的可迭代对象(iterable)。

**process_results(response, results)**

当spider返回结果(item或request)时该方法被调用。 设定该方法的目的是在结果返回给框架核心(framework core)之前做最后的处理， 例如设定item的ID。其接受一个结果的列表(list of results)及对应的response。 其结果必须返回一个结果的列表(list of results)(包含Item或者Request对象)。

### XMLFeedSpider例子(XMLFeedSpider example) ###

该spider十分易用。下边是其中一个例子:

	from scrapy.spiders import XMLFeedSpider
	from myproject.items import TestItem
	
	class MySpider(XMLFeedSpider):
	    name = 'example.com'
	    allowed_domains = ['example.com']
	    start_urls = ['http://www.example.com/feed.xml']
	    iterator = 'iternodes'  # This is actually unnecessary, since it's the default value
	    itertag = 'item'
	
	    def parse_node(self, response, node):
	        self.logger.info('Hi, this is a <%s> node!: %s', self.itertag, ''.join(node.extract()))
	
	        item = TestItem()
	        item['id'] = node.xpath('@id').extract()
	        item['name'] = node.xpath('name').extract()
	        item['description'] = node.xpath('description').extract()
	        return item

简单来说，我们在这里创建了一个spider，从给定的 <font color=red>`start_urls`</font> 中下载feed， 并迭代feed中每个 <font color=red>`item`</font> 标签，输出，并在 Item 中存储有些随机数据。

### CSVFeedSpider ###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spiders.CSVFeedSpider
</td></tr></table>

该spider除了其按行遍历而不是节点之外其他和XMLFeedSpider十分类似。 而其在每次迭代时调用的是 `parse_row()` 。

**delimiter**

在CSV文件中用于区分字段的分隔符。类型为string。 默认为 <font color=red>`','`</font> (逗号)。

**quotechar**

CSV文件每个字段的封闭符号，默认为 <font color=red>`'"'`</font>
(引号)。
**headers**

CSV文件列名的列表。

**parse_row(response, row)**

该方法接收一个response对象及一个以提供或检测出来的header为键的字典(代表每行)。 该spider中，您也可以覆盖 <font color=red>`adapt_response`</font> 及 <font color=red>`process_results`</font> 方法来进行预处理(pre-processing)及后(post-processing)处理。

### CSVFeedSpider例子(CSVFeedSpider example) ###

下面的例子和之前的例子很像，但使用了 `CSVFeedSpider`:

	from scrapy.spiders import CSVFeedSpider
	from myproject.items import TestItem
	
	class MySpider(CSVFeedSpider):
	    name = 'example.com'
	    allowed_domains = ['example.com']
	    start_urls = ['http://www.example.com/feed.csv']
	    delimiter = ';'
	    quotechar = "'"
	    headers = ['id', 'name', 'description']
	
	    def parse_row(self, response, row):
	        self.logger.info('Hi, this is a row!: %r', row)
	
	        item = TestItem()
	        item['id'] = row['id']
	        item['name'] = row['name']
	        item['description'] = row['description']
	        return item

### SitemapSpider ###


<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spiders.SitemapSpider
</td></tr></table>

SitemapSpider使您爬取网站时可以通过 Sitemaps 来发现爬取的URL。

其支持嵌套的sitemap，并能从 robots.txt 中获取sitemap的url。

**sitemap_urls**

包含您要爬取的url的sitemap的url列表(list)。 您也可以指定为一个 robots.txt ，spider会从中分析并提取url。

**sitemap_rules**
一个包含 (regex, callback) 元组的列表(list):

  - <font color=red>`regex`</font> 是一个用于匹配从sitemap提供的url的正则表达式。 <font color=red>`regex`</font> 可以是一个字符串或者编译的正则对象(compiled regex object)。
  - <font color=red>`callback`</font> 指定了匹配正则表达式的url的处理函数。 <font color=red>`callback`</font> 可以是一个字符串(spider中方法的名字)或者是callable。

例如:

	sitemap_rules = [('/product/', 'parse_product')]

规则按顺序进行匹配，之后第一个匹配才会被应用。

如果您忽略该属性，sitemap中发现的所有url将会被 <font color=red>`parse`</font> 函数处理。

**sitemap_follow**

一个用于匹配要跟进的sitemap的正则表达式的列表(list)。其仅仅被应用在 使用 Sitemap index files 来指向其他sitemap文件的站点。

默认情况下所有的sitemap都会被跟进。

**sitemap\_alternate\_links**

指定当一个 <font color=red>`url`</font> 有可选的链接时，是否跟进。 有些非英文网站会在一个 <font color=red>`url`</font> 块内提供其他语言的网站链接。

例如:

	<url>
	    <loc>http://example.com/</loc>
	    <xhtml:link rel="alternate" hreflang="de" href="http://example.com/de"/>
	</url>

当 <font color=red>`sitemap_alternate_links`</font> 设置时，两个URL都会被获取。 当 <font color=red>`sitemap_alternate_links`</font> 关闭时，只有 <font color=red>`http://example.com/`</font> 会被获取。

默认 <font color=red>`sitemap_alternate_links`</font> 关闭。

### SitemapSpider示例(SitemapSpider examples) ###

简单的例子: 使用 <font color=red>`parse`</font> 处理通过sitemap发现的所有url:

	from scrapy.spiders import SitemapSpider
	
	class MySpider(SitemapSpider):
	    sitemap_urls = ['http://www.example.com/sitemap.xml']
	
	    def parse(self, response):
	        pass # ... scrape item here ...

用特定的函数处理某些url，其他的使用另外的callback:

	from scrapy.spiders import SitemapSpider
	
	class MySpider(SitemapSpider):
	    sitemap_urls = ['http://www.example.com/sitemap.xml']
	    sitemap_rules = [
	        ('/product/', 'parse_product'),
	        ('/category/', 'parse_category'),
	    ]
	
	    def parse_product(self, response):
	        pass # ... scrape product ...
	
	    def parse_category(self, response):
	        pass # ... scrape category ...

跟进 robots.txt 文件定义的sitemap并只跟进包含有 <font color=red>`/sitemap_shop`</font> 的url:

	from scrapy.spiders import SitemapSpider
	
	class MySpider(SitemapSpider):
	    sitemap_urls = ['http://www.example.com/robots.txt']
	    sitemap_rules = [
	        ('/shop/', 'parse_shop'),
	    ]
	    sitemap_follow = ['/sitemap_shops']
	
	    def parse_shop(self, response):
	        pass # ... scrape shop here ...

在SitemapSpider中使用其他url:

	from scrapy.spiders import SitemapSpider
	
	class MySpider(SitemapSpider):
	    sitemap_urls = ['http://www.example.com/robots.txt']
	    sitemap_rules = [
	        ('/shop/', 'parse_shop'),
	    ]
	
	    other_urls = ['http://www.example.com/about']
	
	    def start_requests(self):
	        requests = list(super(MySpider, self).start_requests())
	        requests += [scrapy.Request(x, self.parse_other) for x in self.other_urls]
	        return requests
	
	    def parse_shop(self, response):
	        pass # ... scrape shop here ...
	
	    def parse_other(self, response):
	        pass # ... scrape other here ...