# Scrapy终端(Scrapy shell) #

Scrapy终端是一个交互终端，供您在未启动spider的情况下尝试及调试您的爬取代码。 其本意是用来测试提取数据的代码，不过您可以将其作为正常的Python终端，在上面测试任何的Python代码。

该终端是用来测试XPath或CSS表达式，查看他们的工作方式及从爬取的网页中提取的数据。 在编写您的spider时，该终端提供了交互性测试您的表达式代码的功能，免去了每次修改后运行spider的麻烦。

一旦熟悉了Scrapy终端后，您会发现其在开发和调试spider时发挥的巨大作用。

## 配置终端(Configuring the shell) ##

如果你安装了Ipython，Scrapy终端将会使用它(而不是标准python控制台)。Ipython控制台更加强大，在标准控制台的基础上提供了灵巧的自动补全和彩色输出。

我们强烈推荐你安装Ipython，特别是你在Unix系统上工作(这里Ipython擅长)。

Scrapy也支持bpython，Ipython不能用时就会使用它。

通过Scrapy的设置，你可以配置使用 <font color=red>`ipyhton`</font>, <font color=red>`bpyhton`</font> 或者标准 <font color=red>`python`</font>终端，而不用管哪种安装了。设置 <font color=red>`SCRAPY_PYTHON_SHELL`</font> 环境变量；或者在 scrapy.cfg 中定义：

	[settings]
	shell = bpython

## 开启终端(Launch the shell) ##

开启Scrapy终端，你可以使用 `shell` 命令，就像这样：

	scrapy shell <url>

这里的 <font color=red>`url`</font> 是你要爬取的URL。

`shell` 也适用于本地文件。如果你想使用网页的本地副本，这将非常方便，`.shell` 能通过下面的语法了解你要使用本地文件：

	# UNIX-style
	scrapy shell ./path/to/file.html
	scrapy shell ../other/path/to/file.html
	scrapy shell /absolute/path/to/file.html
	
	# File URI
	scrapy shell file:///absolute/path/to/file.html

<font color=#0000ff>
NOTE:</br>当使用相对路径的时候，明确在前面添加 <font color=red>**./**</font> (或者 <font color=red>**../**</font>)。<font color=red>`scrapy shell index.html`</font> 将不会像你期望的那样工作(就是这样设计的，而不是bug)。

因为 `shell` 的HTTP URLs优先级高于FILE URIs，根据语法 <font color=red>`index.html`</font> 与 <font color=red>`example.com`</font> 类似，`shell` 会把 <font color=red>`index.html`</font> 看做一个域名，然后就会引发DNS查找错误。

	$ scrapy shell index.html
	[ ... scrapy shell starts ... ]
	[ ... traceback ... ]
	twisted.internet.error.DNSLookupError: DNS lookup failed:
	address 'index.html' not found: [Errno -5] No address associated with hostname.

`shell` 不会事先查看本地目录中是否存在叫 <font color=red>`index.html`</font> 的文件。在此提醒，明确本地路径。
</font>

# 使用终端(Using the shell) #

Scrapy终端仅仅是一个普通的Python终端(或 IPython )。其提供了一些额外的快捷方式。

## 可用的快捷方式(Available Shortcuts) ##
  - <font color=red>`shelp()`</font> - 打印可用对象及快捷命令的帮助列表
  - <font color=red>`fetch(url[, redirect=True])`</font> -  根据给定的请求(request)或URL获取一个新的response，并更新相关的对象。你可以定义 <font color=red>`redirect=False`</font> 来关闭请求重定向。
  - <font color=red>`fetch(request)`</font> - 从给定的request对象中获取新的response，并更新相关的对象。
  - <font color=red>`view(response)`</font> - 在本机的浏览器打开给定的response。 其会在response的body中添加一个 <base> tag ，使得外部链接(例如图片及css)能正确显示。 注意，该操作会在本地创建一个临时文件，且该文件不会被自动删除。

## 可用的Scrapy对象(Available Scrapy objects) ##

Scrapy终端根据下载的页面会自动创建一些方便使用的对象，例如 Response 对象及 Selector 对象(对HTML及XML内容)。

这些对象有：

  - <font color=red>`crawler`</font> - 当前 Crawler 对象.
  - <font color=red>`spider`</font> - 处理URL的spider。 对当前URL没有处理的Spider时则为一个 Spider 对象。
  - <font color=red>`request`</font> - 最近获取到的页面的 Request 对象。 您可以使用 `replace()` 修改该 request。或者 使用 <font color=red>`fetch`</font>快捷方式来获取新的request。
  - <font color=red>`response`</font> - 包含最近获取到的页面的 Response 对象。
  - <font color=red>`settings`</font> - 当前的 Scrapy settings

## 终端会话例子(Example of shell session) ##

下面给出一个典型的终端会话的例子。 在该例子中，我们首先爬取了 http://scarpy.org 的页面，而后接着爬取 http://slashdot.org 的页面。 最后，我们修改了(Slashdot)的请求，将请求设置为POST并重新获取， 得到HTTP 405(不允许的方法)错误。 之后通过Ctrl-D(Unix)或Ctrl-Z(Windows)关闭会话。

需要注意的是，由于爬取的页面不是静态页，内容会随着时间而修改， 因此例子中提取到的数据可能与您尝试的结果不同。 该例子的唯一目的是让您熟悉Scrapy终端。

首先，我们启动终端:

	scrapy shell 'https://scrapy.org' --nolog

接着该终端(使用Scrapy下载器(downloader))获取URL内容并打印可用的对象及快捷命令(注意到以 <font color=red>`[s]`</font> 开头的行):

	[s] Available Scrapy objects:
	[s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
	[s]   crawler    <scrapy.crawler.Crawler object at 0x7f07395dd690>
	[s]   item       {}
	[s]   request    <GET https://scrapy.org>
	[s]   response   <200 https://scrapy.org/>
	[s]   settings   <scrapy.settings.Settings object at 0x7f07395dd710>
	[s]   spider     <DefaultSpider 'default' at 0x7f0735891690>
	[s] Useful shortcuts:
	[s]   fetch(url[, redirect=True]) Fetch URL and update local objects (by default, redirects are followed)
	[s]   fetch(req)                  Fetch a scrapy.Request and update local objects
	[s]   shelp()           Shell help (print this help)
	[s]   view(response)    View response in a browser
	
	>>>

之后，您就可以操作这些对象了:

	>>> response.xpath('//title/text()').extract_first()
	'Scrapy | A Fast and Powerful Scraping and Web Crawling Framework'
	
	>>> fetch("https://reddit.com")
	
	>>> response.xpath('//title/text()').extract()
	['reddit: the front page of the internet']
	
	>>> request = request.replace(method="POST")
	
	>>> fetch(request)
	
	>>> response.status
	404
	
	>>> from pprint import pprint
	
	>>> pprint(response.headers)
	{'Accept-Ranges': ['bytes'],
	 'Cache-Control': ['max-age=0, must-revalidate'],
	 'Content-Type': ['text/html; charset=UTF-8'],
	 'Date': ['Thu, 08 Dec 2016 16:21:19 GMT'],
	 'Server': ['snooserv'],
	 'Set-Cookie': ['loid=KqNLou0V9SKMX4qb4n; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure',
	                'loidcreated=2016-12-08T16%3A21%3A19.445Z; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure',
	                'loid=vi0ZVe4NkxNWdlH7r7; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure',
	                'loidcreated=2016-12-08T16%3A21%3A19.459Z; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure'],
	 'Vary': ['accept-encoding'],
	 'Via': ['1.1 varnish'],
	 'X-Cache': ['MISS'],
	 'X-Cache-Hits': ['0'],
	 'X-Content-Type-Options': ['nosniff'],
	 'X-Frame-Options': ['SAMEORIGIN'],
	 'X-Moose': ['majestic'],
	 'X-Served-By': ['cache-cdg8730-CDG'],
	 'X-Timer': ['S1481214079.394283,VS0,VE159'],
	 'X-Ua-Compatible': ['IE=edge'],
	 'X-Xss-Protection': ['1; mode=block']}
	>>>

## 在spider中启动shell来查看response(Invoking the shell from spiders to inspect responses) ##

有时您想在spider的某个位置中查看被处理的response， 以确认您期望的response到达特定位置。

这可以通过 <font color=red>`scrapy.shell.inspect_response`</font> 函数来实现。

以下是如何在spider中调用该函数的例子:

	import scrapy
	
	
	class MySpider(scrapy.Spider):
	    name = "myspider"
	    start_urls = [
	        "http://example.com",
	        "http://example.org",
	        "http://example.net",
	    ]
	
	    def parse(self, response):
	        # We want to inspect one specific response.
	        if ".org" in response.url:
	            from scrapy.shell import inspect_response
	            inspect_response(response, self)
	
	        # Rest of parsing code.

当运行spider时，您将得到类似下列的输出:

	2014-01-23 17:48:31-0400 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://example.com> (referer: None)
	2014-01-23 17:48:31-0400 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://example.org> (referer: None)
	[s] Available Scrapy objects:
	[s]   crawler    <scrapy.crawler.Crawler object at 0x1e16b50>
	...
	
	>>> response.url
	'http://example.org'

接着测试提取代码:

	>>> response.xpath('//h1[@class="fn"]')
	[]

呃，看来是没有。您可以在浏览器里查看response的结果，判断是否是您期望的结果:

	>>> view(response)
	True

最后您可以点击Ctrl-D(Windows下Ctrl-Z)来退出终端，恢复爬取:

	>>> ^D
	2014-01-23 17:50:03-0400 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://example.net> (referer: None)
	...

<font color=#0000ff>
NOTE:</br>注意: 由于该终端屏蔽了Scrapy引擎，您在这个终端中不能使用 fetch 快捷命令(shortcut)。 当您离开终端时，spider会从其停下的地方恢复爬取，正如上面显示的那样。
</font>