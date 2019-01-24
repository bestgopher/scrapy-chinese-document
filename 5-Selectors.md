#选择器(Selectors)#

当你爬取网页时，你需要指定的最常见的任务是从HTML源码中提取数据。有几个能实现此功能的库。

 - <font color=green>BeautifulSoup</font> 是在Python程序员中非常受欢迎的网页提取库，它通过HTML代码的结构构造一个Python对象，并同时也处理不完整的标签，但是它有一个缺点:慢。
 - <font color=green>lxml</font> 是一个XML解析库(也解析HTML)，它使用基于元素树(ElementTree)的python风格(pythonic)的API完成的。(lxml不是pyhton标准库)

Scrapy提供了专属的数据解析机制。它们被称为selectors是因为它们'select'通过Xpath或者CSS表达式指定的HTML文档的一部分。

Xpath是一门选择XML文档节点的语言，也可以在HTML中使用。CSS是应用于HTML样式中的语言，它定义的选择器为指定的HTML元素关联样式。

Scrapy选择器(selectors)是在lxml库上建立的，这意味着它们在解析速度和准确的上都和lxml相似。

这章将解释这些选择器怎样使用，描述那些小而简单的API，不像lxml的API那样大，因为lxm库除了选择文档标签之外还能被用来其他工作。

#使用选择器(Using selectors)#

##构造选择器(Constructing selectors)##

Scrapy选择器是用 **text** 或者 **TextResponse** 对象构建的 <font color=red>`Selector`</font> 类的实例。它会根据输入的格式自动选择最好的解析规则(XML vs HTML)。

	>>> from scrapy.selector import Selector
	>>> from scrapy.http import HtmlResponse

通过text构造：

	>>> body = '<html><body><span>good</span></body></html>'
	>>> Selector(text=body).xpath('//span/text()').extract()
	[u'good']

通过响应对象构造：

	>>> response = HtmlResponse(url='http://example.com', body=body)
	>>> Selector(response=response).xpath('//span/text()').extract()
	[u'good']

为了方便，response对象实现了一个 *.selector* 属性，使用这个快捷方式完全OK。

	>>> response.selector.xpath('//span/text()').extract()
	[u'good']

##使用选择器(Using selectors)##

为了解释怎么使用选择器，我们将在 Scrapy shell(提供了交互式测试) 中演示，测试的网页在Scrapy文档服务器中:
https://doc.scrapy.org/en/latest/_static/selectors-sample1.html

这是HTML代码：

	<html>
	 <head>
	  <base href='http://example.com/' />
	  <title>Example website</title>
	 </head>
	 <body>
	  <div id='images'>
	   <a href='image1.html'>Name: My image 1 <br /><img src='image1_thumb.jpg' /></a>
	   <a href='image2.html'>Name: My image 2 <br /><img src='image2_thumb.jpg' /></a>
	   <a href='image3.html'>Name: My image 3 <br /><img src='image3_thumb.jpg' /></a>
	   <a href='image4.html'>Name: My image 4 <br /><img src='image4_thumb.jpg' /></a>
	   <a href='image5.html'>Name: My image 5 <br /><img src='image5_thumb.jpg' /></a>
	  </div>
	 </body>
	</html>

首先，打开shell：
	
	scrapy shell https://doc.scrapy.org/en/latest/_static/selectors-sample1.html

然后，shell加载完毕之后，你将有 <font color=red>`response`</font> 变量绑定的response对象，response对象附加的 <font color=red>`response.selector`</font> 属性使用。

因为我们在处理HTML，所以选择器将会自动使用HTML解析器。

因此，查看这个网页的HTML代码，我们构造一个XPath来提取title标签里的文本内容。

	>>> response.selector.xpath('//title/text()')
	[<Selector (text) xpath=//title/text()>]

使用XPath和CSS查询响应非常常用，因此response对象有两个方便的快捷方式：<font color=red>`response.xpath()`</font> 和 <font color=red>`response.css()`</font>。

	>>> response.xpath('//title/text()')
	[<Selector (text) xpath=//title/text()>]
	>>> response.css('title::text')
	[<Selector (text) xpath=//title/text()>]

正如你看到的，<font color=red>`.xpath`</font> 和 <font color=red>`.css`</font> 方法返回一个 `SelectorList` 实例，是一个 `Selector` 实例的列表。此API可用于快速选择嵌套的数据。

	>>> response.css('img').xpath('@src').extract()
	[u'image1_thumb.jpg',
	 u'image2_thumb.jpg',
	 u'image3_thumb.jpg',
	 u'image4_thumb.jpg',
	 u'image5_thumb.jpg']

要提取文本数据，你必须调用选择器的 <font color=red>·.extract()·</font> 方法，如下所示：

	>>> response.xpath('//title/text()').extract()
	[u'Example website']

如果你想要提取第一个匹配的元素，你可以调用 <font color=red>`.first_exctarct()`</font>

	>>> response.xpath('//div[@id="images"]/a/text()').extract_first()
	u'Name: My image 1 '

如果没有元素满足，它返回 <font color=red>`None`</font>：

	>>> response.xpath('//div[@id="not-exists"]/text()').extract_first() is None
	True

为了返回值不是 <font color=red>`None`</font>， 我们可以在参数中提供一个默认的返回值：

	>>> response.xpath('//div[@id="not-exists"]/text()').extract_first(default='not-found')
	'not-found'

注意，CSS选择器可以使用css3伪元素提取文本或者属性。

	>>> response.css('title::text').extract()
	[u'Example website']

现在我们要得到base URL和一些图片的链接：

	>>> response.xpath('//base/@href').extract()
	[u'http://example.com/']
	
	>>> response.css('base::attr(href)').extract()
	[u'http://example.com/']
	
	>>> response.xpath('//a[contains(@href, "image")]/@href').extract()
	[u'image1.html',
	 u'image2.html',
	 u'image3.html',
	 u'image4.html',
	 u'image5.html']
	
	>>> response.css('a[href*=image]::attr(href)').extract()
	[u'image1.html',
	 u'image2.html',
	 u'image3.html',
	 u'image4.html',
	 u'image5.html']
	
	>>> response.xpath('//a[contains(@href, "image")]/img/@src').extract()
	[u'image1_thumb.jpg',
	 u'image2_thumb.jpg',
	 u'image3_thumb.jpg',
	 u'image4_thumb.jpg',
	 u'image5_thumb.jpg']
	
	>>> response.css('a[href*=image] img::attr(src)').extract()
	[u'image1_thumb.jpg',
	 u'image2_thumb.jpg',
	 u'image3_thumb.jpg',
	 u'image4_thumb.jpg',
	 u'image5_thumb.jpg']

##嵌套选择(Nesting selectors)##

选择器方法 <font color=red>`.xpath()`</font> 和 <font color=red>`.css()`</font> 返回选择器对象的列表，因此你也可以用这些选择器对象调用选择器方法。例如：

	>>> links = response.xpath('//a[contains(@href, "image")]')
	>>> links.extract()
	[u'<a href="image1.html">Name: My image 1 <br><img src="image1_thumb.jpg"></a>',
	 u'<a href="image2.html">Name: My image 2 <br><img src="image2_thumb.jpg"></a>',
	 u'<a href="image3.html">Name: My image 3 <br><img src="image3_thumb.jpg"></a>',
	 u'<a href="image4.html">Name: My image 4 <br><img src="image4_thumb.jpg"></a>',
	 u'<a href="image5.html">Name: My image 5 <br><img src="image5_thumb.jpg"></a>']
	
	>>> for index, link in enumerate(links):
	...     args = (index, link.xpath('@href').extract(), link.xpath('img/@src').extract())
	...     print 'Link number %d points to url %s and image %s' % args
	
	Link number 0 points to url [u'image1.html'] and image [u'image1_thumb.jpg']
	Link number 1 points to url [u'image2.html'] and image [u'image2_thumb.jpg']
	Link number 2 points to url [u'image3.html'] and image [u'image3_thumb.jpg']
	Link number 3 points to url [u'image4.html'] and image [u'image4_thumb.jpg']
	Link number 4 points to url [u'image5.html'] and image [u'image5_thumb.jpg']

##选择器和正则结合(Using selectors with regular expressions)##

**selector** 还有一个 <font color=red>`.re()`</font> 方法，通过正则表达式提取数据。然而，与 <font color=red>`.xpath()`</font> 和 <font color=red>`.css()`</font> 方法不同， <font color=red>`.re()`</font> 返回一个unicode字符串的列表。因此你不能构建嵌套式的 <font color=red>`.re()`</font> 调用。

这是一个从上面的HTML代码提取图片名称的例子：
	
	>>> response.xpath('//a[contains(@href, "image")]/text()').re(r'Name:\s*(.*)')
	[u'My image 1',
	 u'My image 2',
	 u'My image 3',
	 u'My image 4',
	 u'My image 5']

这里为 <font color=red>`.re()`</font> 额外添加了一个类似于 <font color=red>`extract\_first()`</font> 的方法，叫 <font color=red>`re_first()`</font>。用这个提取第一个匹配的字符串。

	>>> response.xpath('//a[contains(@href, "image")]/text()').re_first(r'Name:\s*(.*)')
	u'My image 1'

##使用相对xpath(Working with relative XPahts)##

记住，你调用内嵌式选择器时XPath以 <font color=red>`/`</font> 开始，Xpath将从文档的绝对路径而不是相对路径开始选择。

比如，假设你想要提取 <font color=red>`<div>`</font> 元素中所有的 <font color=red>`<p>`</font> 元素。首先，你应该获得所有的 <font color=red>`<div>`</font> 元素:

	>>> divs = response.xpath('//div')

起初，你可能会使用下面的方法，它是错的，实际上它会提取文档中所有的 <font color=red>`<p>`</font> 元素，而不仅是 <font color=red>`<div>`</font> 中的。

	>>> for p in divs.xpath('//p'):  # this is wrong - gets all <p> from the whole document
	...     print p.extract()

下面是比较合适的处理方法(注意Xpath表达式 <font color=red>`.//p`</font> 中的 <font color=red>`.`</font> 前缀):

	>>> for p in divs.xpath('.//p'):  # extracts all <p> inside
	...     print p.extract()


另一种常见情况是提取直接后代 <font color=red>`<p>`</font> 元素：
	
	>>> for p in divs.xpath('p'):
	...     print p.extract()

##XPath表达式中的变量(Variables in XPath expressions)##

XPath允许在XPath中引用变量，语法是 <font color=red>`$somevariable`</font>。这就像SQL中的参数化查询或者预准备语句，你可以在使用占位符 <font color=red>`?`</font> 的地方用一些参数替换，这样就可以在查询用用这些值替换。

下面是一个例子，我们通过"id"属性来匹配一个元素，而不对其硬编码(就像我们前面所示的一样):

	>>> # `$val` used in the expression, a `val` argument needs to be passed
	>>> response.xpath('//div[@id=$val]/a/text()', val='images').extract_first()
	u'Name: My image 1 '

这是另一个例子，提取一个包含5个 <font color=red>`<a>`</font> 子元素的 <font color=red>`<div>`</font> 的'id'属性(这里我们将值 <font color=red>`5`</font> 作为整数传递)：

	>>> response.xpath('//div[count(a)=$cnt]/@id', cnt=5).extract_first()
	u'images'

当调用 <font color=red>`.path()`</font> 时，所有的变量都必须绑定一个值(否则你将会得到 <font color=red>`ValueError：XPath error：`</font> 异常)。这就意味着传递一样数量的参数是必要的。

<font color=red>`parsel`</font>，一个强大的Scrapy选择器的库，有更多的细节和例子演示Xpath使用变量。

#使用EXSLT扩展(Using EXSLT extensions)#

因为Scrapy selectors是建立在lxml之上的，因此也支持一些EXSLT扩展，可以在XPath表达式中使用这些预先制定的命名空间：

<table>
<tr>
<td>prefix</td>
<td>namespace</td>
<td>usage</td>
</tr>
<tr>
<td>re</td>
<td>http://exslt.org/regular-expressions</td>
<td>regular expressions</td>
</tr>
<tr>
<td>set</td>
<td>http://exslt.org/sets</td>
<td>set manipulation</td>
</tr>
</table>

##正则表达式(Regular expressions)##

当 <font color=red>`starts-with()`</font> 或者 <font color=red>`contains()`</font> 不能满足要求时，<font color=red>`test()`</font> 函数能够提供很大的作用。

举例，提取下面'class'属性为数字结尾的链接：

	>>> from scrapy import Selector
	>>> doc = """
	... <div>
	...     <ul>
	...         <li class="item-0"><a href="link1.html">first item</a></li>
	...         <li class="item-1"><a href="link2.html">second item</a></li>
	...         <li class="item-inactive"><a href="link3.html">third item</a></li>
	...         <li class="item-1"><a href="link4.html">fourth item</a></li>
	...         <li class="item-0"><a href="link5.html">fifth item</a></li>
	...     </ul>
	... </div>
	... """
	>>> sel = Selector(text=doc, type="html")
	>>> sel.xpath('//li//@href').extract()
	[u'link1.html', u'link2.html', u'link3.html', u'link4.html', u'link5.html']
	>>> sel.xpath('//li[re:test(@class, "item-\d$")]//@href').extract()
	[u'link1.html', u'link2.html', u'link4.html', u'link5.html']
	>>>

<font color=#FF7F24>
Warning:
</br>C语言库 libxslt 不原生支持EXSLT正则表达式，因此 lxml 在实现时使用了Python re 模块的钩子。 因此，在XPath表达式中使用regexp函数可能会牺牲少量的性能。
</font>

##集合操作(Set operations)##

集合操作可以方便地用于在提取文字元素前从文档树中去除一些部分。

例如使用itemscopes组和对应的itemprops来提取微数据(来自http://schema.org/Product的样本内容):

	>>> doc = """
	... <div itemscope itemtype="http://schema.org/Product">
	...   <span itemprop="name">Kenmore White 17" Microwave</span>
	...   <img src="kenmore-microwave-17in.jpg" alt='Kenmore 17" Microwave' />
	...   <div itemprop="aggregateRating"
	...     itemscope itemtype="http://schema.org/AggregateRating">
	...    Rated <span itemprop="ratingValue">3.5</span>/5
	...    based on <span itemprop="reviewCount">11</span> customer reviews
	...   </div>
	...
	...   <div itemprop="offers" itemscope itemtype="http://schema.org/Offer">
	...     <span itemprop="price">$55.00</span>
	...     <link itemprop="availability" href="http://schema.org/InStock" />In stock
	...   </div>
	...
	...   Product description:
	...   <span itemprop="description">0.7 cubic feet countertop microwave.
	...   Has six preset cooking categories and convenience features like
	...   Add-A-Minute and Child Lock.</span>
	...
	...   Customer reviews:
	...
	...   <div itemprop="review" itemscope itemtype="http://schema.org/Review">
	...     <span itemprop="name">Not a happy camper</span> -
	...     by <span itemprop="author">Ellie</span>,
	...     <meta itemprop="datePublished" content="2011-04-01">April 1, 2011
	...     <div itemprop="reviewRating" itemscope itemtype="http://schema.org/Rating">
	...       <meta itemprop="worstRating" content = "1">
	...       <span itemprop="ratingValue">1</span>/
	...       <span itemprop="bestRating">5</span>stars
	...     </div>
	...     <span itemprop="description">The lamp burned out and now I have to replace
	...     it. </span>
	...   </div>
	...
	...   <div itemprop="review" itemscope itemtype="http://schema.org/Review">
	...     <span itemprop="name">Value purchase</span> -
	...     by <span itemprop="author">Lucas</span>,
	...     <meta itemprop="datePublished" content="2011-03-25">March 25, 2011
	...     <div itemprop="reviewRating" itemscope itemtype="http://schema.org/Rating">
	...       <meta itemprop="worstRating" content = "1"/>
	...       <span itemprop="ratingValue">4</span>/
	...       <span itemprop="bestRating">5</span>stars
	...     </div>
	...     <span itemprop="description">Great microwave for the price. It is small and
	...     fits in my apartment.</span>
	...   </div>
	...   ...
	... </div>
	... """
	>>> sel = Selector(text=doc, type="html")
	>>> for scope in sel.xpath('//div[@itemscope]'):
	...     print "current scope:", scope.xpath('@itemtype').extract()
	...     props = scope.xpath('''
	...                 set:difference(./descendant::*/@itemprop,
	...                                .//*[@itemscope]/*/@itemprop)''')
	...     print "    properties:", props.extract()
	...     print
	
	current scope: [u'http://schema.org/Product']
	    properties: [u'name', u'aggregateRating', u'offers', u'description', u'review', u'review']
	
	current scope: [u'http://schema.org/AggregateRating']
	    properties: [u'ratingValue', u'reviewCount']
	
	current scope: [u'http://schema.org/Offer']
	    properties: [u'price', u'availability']
	
	current scope: [u'http://schema.org/Review']
	    properties: [u'name', u'author', u'datePublished', u'reviewRating', u'description']
	
	current scope: [u'http://schema.org/Rating']
	    properties: [u'worstRating', u'ratingValue', u'bestRating']
	
	current scope: [u'http://schema.org/Review']
	    properties: [u'name', u'author', u'datePublished', u'reviewRating', u'description']
	
	current scope: [u'http://schema.org/Rating']
	    properties: [u'worstRating', u'ratingValue', u'bestRating']
	
	>>>

在这里，我们首先在 itemscope 元素上迭代，对于其中的每一个元素，我们寻找所有的 itemprops 元素，并排除那些本身在另一个 itemscope 内的元素。

#一些XPath小技巧(some XPath tips)#

根据这篇文章(https://blog.scrapinghub.com/2014/07/17/xpath-tips-from-the-web-scraping-trenches)，在通过Scrapy选择器使用XPath时有一些很好用的小技巧。如果你对XPath不是很熟悉，你可以先看看XPath教程。

##在条件中使用文本节点(Using text nodes in a condition)##

当你需要通过Xpath字符串函数提取文本内容时，避免使用 <font color=red>`.//text()`</font>，而是使用 <font color=red>`.`</font> 代替。

这是因为 <font color=red>`.//text()`</font> 表达式生成一组文本元素 - 一个节点集。当这个表达式作为参数传递给字符串函数，例如 <font color=red>`contains()`</font> 或者 <font color=red>`starts-with()`</font>，这个节点集被转化为字符串，但是转化的结果文本只是这个节点集的第一个元素而已。

例如：

	>>> from scrapy import Selector
	>>> sel = Selector(text='<a href="#">Click here to go to the <strong>Next Page</strong></a>')

转换节点集为字符串：

	>>> sel.xpath('//a//text()').extract() # take a peek at the node-set
	[u'Click here to go to the ', u'Next Page']
	>>> sel.xpath("string(//a[1]//text())").extract() # convert it to string
	[u'Click here to go to the ']

然而，一个节点转为字符串，把自身的文本和后代的文本合在一起：

	>>> sel.xpath("//a[1]").extract() # select the first node
	[u'<a href="#">Click here to go to the <strong>Next Page</strong></a>']
	>>> sel.xpath("string(//a[1])").extract() # convert it to string
	[u'Click here to go to the Next Page']

因此，在这个例子中使用 <font color=red>`.//text()`</font> 不能提取任何内容：

	>>> sel.xpath("//a[contains(.//text(), 'Next Page')]").extract()
	[] 

但是使用 <font color=red>`.`</font> 意味着整个节点，能达到效果：

	>>> sel.xpath("//a[contains(., 'Next Page')]").extract()
	[u'<a href="#">Click here to go to the <strong>Next Page</strong></a>']

##警惕 //node[1] 和 (//node)[1] 的区别##

<font color=red>`//node[1]`</font> 提取的是每个父节点下的第一个节点。

<font color=red>`(//node)[1]`</font> 提取的是文档中所有节点的第一个节点。

	>>> from scrapy import Selector
	>>> sel = Selector(text="""
	....:     <ul class="list">
	....:         <li>1</li>
	....:         <li>2</li>
	....:         <li>3</li>
	....:     </ul>
	....:     <ul class="list">
	....:         <li>4</li>
	....:         <li>5</li>
	....:         <li>6</li>
	....:     </ul>""")
	>>> xp = lambda x: sel.xpath(x).extract()


这个提取所有第一个 <li color=red>`<li>`</font> 元素，不论它的父节点是什么：

	>>> xp("//li[1]")
	[u'<li>1</li>', u'<li>4</li>'] 

这个提取整个文档中第一个 <font color=red>`<li>`</font> 元素：

	>>> xp("//ul/li[1]")
	[u'<li>1</li>', u'<li>4</li>']

这个提取整个文档中父节点为 <font color=red>`<ul>`</font> 的第一个 <font color=red><li\></font> 元素。

	>>> xp("(//ul/li)[1]")
	[u'<li>1</li>']

##当通过class属性提取元素，考虑使用CSS##

因为一个元素能有多个class属性，因此使用XPath通过class属性提取元素就显得冗长：

	*[contains(concat(' ', normalize-space(@class), ' '), ' someclass ')]

如果你使用 <font color=red>`@class="someclass"`</font>，你可能最终漏掉有其他class属性的元素。如果你使用 <font color=red>`contains(@class, "someclass")`</font> ，最终你可能得到比你想要数量更多的元素，因为它们有不同的class名称，但是都共有 <font color=red>`someclass`</font>。

当出现这种问题，Scrapy选择器允许使用链式选择器，因此许多情况下你可以通过CSS选择器通过class名提取数据，在需要使用XPath的时候转为XPath提取：

	>>> from scrapy import Selector
	>>> sel = Selector(text='<div class="hero shout"><time datetime="2014-07-23 19:00">Special date</time></div>')
	>>> sel.css('.shout').xpath('./time/@datetime').extract()
	[u'2014-07-23 19:00']

这样比上面显示的使用冗长的XPath更加清晰。只需要记住在后面的XPath表达式中使用 <font color=red>**.**</font>。

#内建选择器参考#

##Selector类##

<table>
<tr>
<td>
<font color=green>class</font> scrapy.selector.Selector<font color=green>(response=None, text=None, type=None)</font>
</td>
</tr>
</table>

**Selector** 实例是封装响应，用来选择响应文本的某一部分。

<font color=red>`response`</font> 是 **HtmlResponse** 或者 **XmlResponse** 对象，它将会被用来选择和提取数据。

<font color=red>`text`</font> 是当 <font color=red>`response`</font>不可用这种情况发生时，选择器提取的文本内容，是一个unicode字符串或者utf-8编码的文本。<font color=red>`text`</font> 和 <font color=red>`response`</font> 一起使用是一种未定义的行为。

<font color=red>`type`</font> 定义选择器的类型，它可以是 <font color=red>`"html"`</font>，<font color=red>`"xml"`</font>，或者 <font color=red>`None`</font>（默认）。

 - 如果 <font color=red>`type`</font> 是 <font color=red>`None`</font>，选择会根据 <font color=red>`response`</font> 的类型自动选择最好的 <font color=red>`type`</font> 值，或者提供了 <font color=red>`text`</font> 时默认为 <font color=red>`"html"`</font>。
 - 如果 <font color=red>`type`</font> 是 <font color=red>`None`</font>，而且提供了 <font color=red>`response`</font>， 选择器会按如下规则推导出类型：
 
    1. **`HtmlResponse`** 为 <font color=red>`"html"`</font> 类型
    2. **`XmlResponse`** 为 <font color=red>`"xml"`</font> 类型
    3. **`其他所有类型`** 为 <font color=red>`"html"`</font> 类型
 
其他情况下，如果设定了 <font color=red>`type`</font> ，选择器类型将被强制设定，而不进行检测。

<font color=green>`xpath`(query)</font>：选择所有满足XPath <font color=red>`query`</font> 语句的节点，返回的结果是单一化的 **`SelectorList`** 实例。列表中的元素实现了 **`Selector`** 的接口。
<font color=red>`query`</font>是包含XPATH查询请求的字符串。注意:为了方便，这个方法也可以这样调用 <font color=red>`response.xpath()`</font>。

<font color=green>`css(query)`</font>：应用CSS选择器选择，返回 **`SelectorList`** 的实例。<font color=red>`query`</font> 是CSS选择器语句的字符串。在后台，CSS查询语句通过 *cssselect* 库被翻译为XPath语句，然后运行 <font color=red>`.xpath()`</font> 方法。注意:为了方便，这个方法也可以这样调用 <font color=red>`response.css()`</font>。

<font color=green>`extract()`</font>：序列化匹配结果，返回匹配节点的unicode字符串组成的列表。结尾是编码内容的百分比。

<font color=green>`re(regex)`</font>：应用给定的正则表达式选择，返回符合条件的unicode字符串。<font color=red>`regex`</font> 可以为编译好的正则表达式或者一个能被 <font color=red>`re.compile(regex)`</font> 编译为正则表达式的字符串。注意：<font color=red>`re()`</font> 和 <font color=red>`re_first()`</font> 会解码HTML实体(除了<font color=red>`&lt`;</font> 和 <font color=red>`&amp;`</font>)

<font color=green>`register_namespace(prefix, uri)`</font>：注册给定的命名空间给 **Selector** 使用。没有注册命令空间，你不能从不标准的命名空间中选择或者提取数据。例子如下。

<font color=green>`remove_namespace()`</font>：移除所有的命名空间。允许少量命名空间的XPath遍历文档。参见下面的例子。

<font color=green>`__nonzero__()`</font>：如果选择了任意的真实文档，将返回 <font color=red>`True`</font> ，否则返回 <font color=red>`False`</font> 。 也就是说， **Selector** 的布尔值是通过它选择的内容确定的。

##SelectorList 对象##


<table>
<tr>
<td>
<font color=green>class</font> scrapy.selector.SelectorList<font color=green>(response=None, text=None, type=None)</font>
</td>
</tr>
</table>

`SelectorList` 是内建的 <font color=red>`list`</font> 的子类，它还有些额外的方法。

<font color=green>`xpath(query)`</font>：为列表中的每个元素调用 <font color=red>`.xpath()`</font> 方法，返回所得结果组成另一个 `SelectorList`。</font color=red>`query`</font> 和 `Selector.xpath()`的参数一致。

<font color=green>`css(query)`</font>：为列表中的每个元素调用 <font color=red>`.css()`</font> 方法，返回所得结果组成另一个 `SelectorList`。<font color=red>`query`</font> 和 `Selector.css()`的参数一致。

<font color=green>`extract()`</font>：为列表中的每个元素调用 <font color=red>`.extract()`</font> 方法，返回unicode字符串组成的列表。

<font color=green>`re()`</font>：为列表中的每个元素调用 <font color=red>`.re()`</font> 方法，返回unicode字符串组成的列表。

##通过HTML响应演示Selector##

这有两三个例子来阐明几个概念。在所有的示例中，我们假设有一个已经通过 `HtmlResponse` 实例化的 `Selector` 对象，就像：
	
	sel = Selector(html_response)

1.从HTML响应体中提取所有的 <font color=red>`<h1>`</font> 元素，返回一个 `Selector` 对象的列表(即 `SelectorList` 对象)：
	
	sel.xpath("//h1")

2.从HTML响应体中提取所有<font color=red>`<h1>`</font> 元素中的文本内容，返回unicode字符串的列表：

	sel.xpath("//h1").extract()         # this includes the h1 tag
	sel.xpath("//h1/text()").extract()  # this excludes the h1 tag 

3.迭代所有的 <font color=red>`<p>`</font> 标签，打印它们的class属性:

	for node in sel.xpath("//p"):
	    print node.xpath("@class").extract() 

##通过XML响应演示Selector##

这有两三个例子来阐明几个概念。在所有的示例中，我们假设有一个已经通过 `XmlResponse` 实例化的 `Selector` 对象，就像：

	sel = Selector(xml_response)


1.从XML响应体中提取所有的 <font color=red>`<product>`</font> 元素，返回一个 `Selector` 对象的列表(即 `SelectorList` 对象)：

	sel.xpath("//product")

2.从 Google Base XML feed 中提取所有的价钱，这需要注册一个命名空间:

	sel.register_namespace("g", "http://base.google.com/ns/1.0")
	sel.xpath("//g:price").extract()

##移除命名空间##

在处理爬虫项目时，完全去掉命名空间而仅仅处理元素名字，写更多简单/实用的XPath会方便很多。你可以为此使用 Selector.remove_namespaces() 方法。

让我们来看一个例子，以Github博客的atom订阅来解释这个情况。

首先，我们使用想爬取的url来打开shell:

	$ scrapy shell https://github.com/blog.atom	

一旦进入shell，我们可以尝试选择所有的 <link> 对象，可以看到没有结果(因为Atom XML命名空间混淆了这些节点):

	>>> response.xpath("//link")
	[]

但一旦我们调用 Selector.remove_namespaces() 方法，所有的节点都可以直接通过他们的名字来访问:

	>>> response.selector.remove_namespaces()
	>>> response.xpath("//link")
	[<Selector xpath='//link' data=u'<link xmlns="http://www.w3.org/2005/Atom'>,
	 <Selector xpath='//link' data=u'<link xmlns="http://www.w3.org/2005/Atom'>,
	 ...

如果你对为什么命名空间移除操作并不总是被调用，而需要手动调用有疑惑。这是因为存在如下两个原因，按照相关顺序如下：

 1. 移除命名空间需要迭代并修改文件的所有节点，而这对于Scrapy爬取的所有文档操作需要一定的性能消耗
 2. 会存在这样的情况，确实需要使用命名空间，但有些元素的名字与命名空间冲突。尽管这些情况非常少见。