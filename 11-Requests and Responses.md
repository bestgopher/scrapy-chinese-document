# Requests and Responses #

Scrapy使用 `Request` 和 `Response` 对象爬取web站点。

一般来说，`Request` 对象在spiders中被生成并且最终传递到 下载器(Downloader)，下载器对其进行处理并返回一个 `Response` 对象， `Response` 对象还会返回到生成request的spider中。

所有 `Request` 和 `Response` 的子类都会实现一些在基类中非必要的 功能。它们会在 Request subclasses 和 Response subclasses 两部分进行详细的说明。

## Request对象(Request objects) ##

<table>
<tr>
<td>
<font color=green>class</font> scrapy.http.Request(url[, callback, method='GET', headers, body, cookies, meta, encoding='utf-8', priority=0, dont_filter=False, errback, flags])
</td>
</tr>
</table>

一个 `Request` 对象代表一个HTTP请求，一般来讲， HTTP请求是由Spider产生并被Downloader处理进而生成一个 `Response`。

参数:</br>
  
  - **url**(string) - 请求的URL
  - **callback**(可调用的) - 这个函数将会被调用，并且这个请求得到的响应作为这个函数的第一个参数(一旦响应被下载)。如果响应没有指定 `callback`，spider的 `parse()` 方法将会被用作回调函数。记住，如果在运行中有异常爬虫，`errback` 将会被调用。
  - **method**(string) - 此请求的HTTP方法。默认是 <font color=red>`'GET'`</font>。
  - **meta**(dict) - `Request.meta` 属性的初始值。 一旦此参数被设置， 通过参数传递的字典将会被浅拷贝。
  - **body**(string或者unicode) - request请求体。如果传进的参数是 <font color=red>`unicode`</font> 类型，将会被编码为 <font color=red>`str`</font> 类型(默认为<font color=red>`utf-8`</font>)。如果 <font color=red>`body`</font> 参数没有给定，那么将会存储一个空的string类型，不管这个参数是什么类型的，最终存储的都会是 <font color=red>`str`</font> 类型(永远不会是 <font color=red>`unicode`</font> 或是 <font color=red>`None`</font>)。
  - **header**(dict) - 请求头。字典值的类型可以是strings (for single valued headers) 或是 lists (for multi-valued headers)。如果传进的值是 <font color=red>`None`</font> ，那么HTTP头将不会被发送。
  - **cookies**(dict或者list) - 请求的cookies。可以被设置成如下两种形式：
     
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用字典：

	request_with_cookies = Request(url="http://www.example.com",
	                               cookies={'currency': 'USD', 'country': 'UY'})

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用列表：

	request_with_cookies = Request(url="http://www.example.com",
	                               cookies=[{'name': 'currency',
	                                        'value': 'USD',
	                                        'domain': 'example.com',
	                                        'path': '/currency'}])
	# name，cookie的键名，
	# value, cookie的值
	# cookie的域名，
	# path, url路径

后者允许自定义cookie的<font color=red>`domain`</font> 和 <font color=red>`path`</font>属性。这仅在保存cookie给后面的请求使用时有用。

当一些网站返回cookies(response中)，这个返回的cookies将会保存到那个域名的cookies中，后面的请求将会发送它们。这是许多常规浏览器的特定行为。然而，如果因为某些原因，你不想与已存在的cookies合并，你可以在 `Request.meta`中设置 <font color=red>`dont_merge_cookies`</font> 为 `True`来定义Scrapy。

  - **encoding**(string) - 请求的编码(默认为<font color=red>`'utf-8'`</font>)。这个编码将会被用作URL的百分号编码的编码方式，也会转换请求体(body)为 <font color=red>`str`</font>(如果给定了<font color=red>`unicode`</font>)
  - **priority**(int) - 这个请求的优先级(默认为<font color=red>`0`</font>)。这个优先级在调度器定义处理请求的顺序时被用到。Requests的优先级越高就越早被执行。允许使用负数表示相对较低的优先级。
  - **dont_filter**(boolean) - 表明请求不能被调度器过滤。当你想执行同一个请求多次，忽略去重过滤，这个参数将会被用到。小心使用它，否则你会进入爬取循环中。默认为 <font color=red>`False`</font>
  - **errback**(可调用的) - 在处理器请求时发生任何异常将会调用此函数。包括类似于HTTP404那样的错误。它接受 Twisted Failure 实例作为第一个参数。
  - **flags(list) - 发送给请求的标志，在日志记录或者类似目的时被使用。

**url**：</br>
  一个包含这个请求URL的字符串。注意这个属性包含转义字符的url，因此它可能与构造函数中传递的url不同。</br>
  这个属性是只读的。要改变Request中的URL使用 `replace()` 方法。

**method**：</br>
  表示HTTP请求方式的字符串。为大写的。例如：<font color=red>`GET`</font>，<font color=red>`POST`</font>，<font color=red>`PUT`</font>，等等。

**headers**：</br>
  包含请求头的类字典的对象。

**body**：</br>
  包含请求体的字符串。</br>这个属性只读。修改请求体使用 `replace()` 方法。

**meta**：</br>
  包含此请求任意元数据的字典。这个字典对于新请求是空的，通常被不同的Scrapy组件(扩展，中间件等)填充。因此这个字典中包含的数据取决于你启用的扩展。</br>当请求被 <font color=red>`copy()`</font> 和 <font color=red>`replace()`</font> 克隆的时候，这个字典是浅拷贝，在我们的spider中，可以使用 <font color=red>response.meta</font> 存取元素。

**copy()**：</br>
  返回通过此请求复制的一个新的请求。

**replace([url, method, headers, body, cookies, meta, encoding, dont_filter, errback, callback])**：</br>
 返回一个具有相同成员的Request对象，除了这些特定成员中不论哪一个指定了新的值之外。属性 `Request.meta` 默认情况下被复制(除非在 <font color=red>`meta`</font> 参数中给定了新的值)。

##传递额外数据到callback函数中##

请求的callback是一个函数，当request的响应被下载时，这个函数被调用。这个回调函数调用时将使用 `Response` 作为第一个参数。

例如：

	def parse_page1(self, response):
	    return scrapy.Request("http://www.example.com/some_page.html",
	                          callback=self.parse_page2)
	
	def parse_page2(self, response):
	    # this would log http://www.example.com/some_page.html
	    self.logger.info("Visited %s", response.url)

某些情况下，你可能需要将某些参数传递给这些回调函数，以便在第二个回调函数中接受这些参数。你可以使用 `Request.meta` 属性解决这个问题。

这里有个示例演示了怎么使用这个机制传递一个item，通过不同的网页填充item不同的字段。

	def parse_page1(self, response):
	    item = MyItem()
	    item['main_url'] = response.url
	    request = scrapy.Request("http://www.example.com/some_page.html",
	                             callback=self.parse_page2)
	    request.meta['item'] = item
	    yield request
	
	def parse_page2(self, response):
	    item = response.meta['item']
	    item['other_url'] = response.url
	    yield item

##使用errback捕获请求处理中的异常(Using errbacks to catch exceptions in request processing)##

请求的errback函数会在处理请求时抛出异常时调用。

它接受 Twisted Failure 实例作为第一个参数，可用于跟踪链接超时，DNS错误等。

这里是一个示例spider，记录所有错误的日志信息，捕获一些特定的错误。

	import scrapy
	
	from scrapy.spidermiddlewares.httperror import HttpError
	from twisted.internet.error import DNSLookupError
	from twisted.internet.error import TimeoutError, TCPTimedOutError
	
	class ErrbackSpider(scrapy.Spider):
	    name = "errback_example"
	    start_urls = [
	        "http://www.httpbin.org/",              # HTTP 200 expected
	        "http://www.httpbin.org/status/404",    # Not found error
	        "http://www.httpbin.org/status/500",    # server issue
	        "http://www.httpbin.org:12345/",        # non-responding host, timeout expected
	        "http://www.httphttpbinbin.org/",       # DNS error expected
	    ]
	
	    def start_requests(self):
	        for u in self.start_urls:
	            yield scrapy.Request(u, callback=self.parse_httpbin,
	                                    errback=self.errback_httpbin,
	                                    dont_filter=True)
	
	    def parse_httpbin(self, response):
	        self.logger.info('Got successful response from {}'.format(response.url))
	        # do something useful here...
	
	    def errback_httpbin(self, failure):
	        # log all failures
	        self.logger.error(repr(failure))
	
	        # in case you want to do something special for some errors,
	        # you may need the failure's type:
	
	        if failure.check(HttpError):
	            # these exceptions come from HttpError spider middleware
	            # you can get the non-200 response
	            response = failure.value.response
	            self.logger.error('HttpError on %s', response.url)
	
	        elif failure.check(DNSLookupError):
	            # this is the original request
	            request = failure.request
	            self.logger.error('DNSLookupError on %s', request.url)
	
	        elif failure.check(TimeoutError, TCPTimedOutError):
	            request = failure.request
	            self.logger.error('TimeoutError on %s', request.url)

## Request.meta特殊的键(Request.meta special keys) ##

`Request.meta` 属性可以包含任意数据，但是有一些特别的键能被Scrapy识别，作为其内置的扩展。

这些：
	
  - `dont_redirect`
  - `dont_retry`
  - `handle_httpstatus_list`
  - `handle_httpstatus_all`
  - `dont_merge_cookies`
  - `cookiejar`
  - `dont_cache`
  - `redirect_urls`
  - `bindaddress`
  - `dont_obey_robotstxt`
  - `download_timeout`
  - `download_maxsize`
  - `download_latency`
  - `download_fail_on_dataloss`
  - `proxy`
  - <font color=red>`ftp_user`</font> (See `FTP_USER` for more info)
  - <font color=red>`ftp_password`</font> (See `FTP_PASSWORD`for more info)
  - `referrer_policy`
  - `max_retry_times`

## bindaddress ##
作为执行请求的传出地址的IP

## download_timeout ##
下载器超时之前的等待时间(s)。只为单个请求设置超时等待时间。详情查看setting中<font color=red>`DOWNLOAD_TIMEOUT`</font>

## download_latency ##
请求开始到接受请求花费的时间，即HTTP消息在网络中传输的时间(请求到获取响应所花费的时间)。这个meta键只有在下载完毕response才能生效。虽然大多数meta键用作控制Scrapy的行为，这个我认为是只读的。

## download\_fail\_on\_dataloss ##
 用来控制当接收到的 response 头信息中的 Content-Length 和内容不匹配或者response chunk 未正确结束时的时所采取的操作。详情查看setting中<font color=red>`DOWNLOAD_FAIL_ON_DATALOSS`</font>

## max\_retry\_times ##
设置重复请求的次数。初始化时，meta键`max_retry_times`的优先级高于设置中的<font color=red>`RETRY_TIMES`</font>

# Request子类(Request subclasses) #

这里列出了内置的 `Request` 子类。你也可以自己定义子类以实现你想要的功能。

## FormRequest对象(FormRequest objects) ##
FormRequest类在 `Request` 的基础上扩展了解决HTML表单的功能。它使用 `lxml.html` 的表单的方法从 `Response` 对象中提取数据，预构建表单的字段。

<table>
<tr>
<td>
<font color=green>class</font> scrapy.http.FormRequest(url[, formdata,callback, method, headers, body, cookies, meta, encoding='utf-8', priority=0, dont_filter=False, errback, flags])
</td>
</tr>
</table>

`FormRequest` 在构造函数中增加了一个新的参数。剩下的参数和 `Request` 类的一样，这里就不列出来了。

**参数**：</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**formdata**(dict或者元组组成的可迭代对象) - 这是一个字典(或者一个(键,值)元组组成的可迭代对象)，包含HTML的表单的数据，这些数据将会被 url-encoded，然后声明在请求体中。

`FormRequest` 对象支持下面除了 `Request` 的标准方法外额外的类方法：

<table><tr><td>
@classmethod</br> 
<font color=red>from_response</font>(response[, formname=None, formid=None, formnumber=0, formdata=None, formxpath=None, formcss=None, clickdata=None, dont_click=False, ...])
</td></tr></table>


返回一个新的 `FormRequest` 对象，这个对象表单的字段是给定的response的HTML中<font color=red>`<form>`</font>元素包含的字段预填充的。

自动模拟点击的策略，默认情况下，应用在看起来能够点击的表单控件上，例如 <font color=red>`<input type='submit'>`</font>。即使这很方便，也很希望有这种行为，但是有时候也会造成一些难以调试的问题。例如，当表单填写完毕后是否使用javascript提交，`from_response()` 默认的行为就不是最合适的。禁用这个行为你可以设置 <font color=red>`dont_click`</font> 为 <font color=red>`True`</font>。而且，如果你想改变点击的行为(而不是禁用它)，你可以使用 <font color=red>`clickdata`</font> 参数。

<font color=orange>
警示：</br>
使用这个方法来提取元素，如果选项值前后存在空格的话，因为 lxml 的原因，将不能正常工作，但是在 lxml3.8 中，以上问题得到的修复。
</font>

**参数**：

  - **response** (`Response` object) - 包含HTML表单的响应，将会用这个表单预填充请求的表单字段。
  - **formname** (string) - 如果给定，name属性为此值的表单将被使用。
  - **formid** (string) - 如果给定，id属性为此值的表单将被使用
  - **formxpath** (string) - 如果给定，xpath匹配到的第一个表单将被使用
  - **formcss** (string) - 如果给定，css匹配到的第一个表单将被使用。
  - **formnumber** (integer) - 当响应包含多个表单，使用第几个表单。默认使用第一个表单，值为 <font color=red>`0`</font>。
  - **formdata** (dict) - 表单中要覆盖的字段。如果一个字段在响应的 <font color=red>`form`</font> 元素中已经出现，这个值将会被这个参数传递进来的值覆盖。如果通过这个参数传递的值为 <font color=red>`None`</font>，request中将不会包含这个字段，即使响应的 <font color=red>`<form>`</font> 元素中存在这个字段。
  - **clickdata** (dict) - 寻找点击控件的属性。如果没有给定，表单数据将会通过模拟点击第一个可点击元素提交。除了html属性之外，表单中的可提交的input标签控件能被从0开始建立索引，使用 <font color=red>`nr`</font> 属性指定索引值。
  - **dont_click**(boolean) - if <font color=red>`True`</font>， 表单数据将会在没有点击任元素的情况下提交。

这个类方法的其他参数将会被直接传递到 `FormRequest` 的构造函数中。

# Request使用实例(Request usage examples) #

## 使用FormRequest通过HTTP POST方法发送数据(Using FormRequest to send data via HTTP POST) ##

如果你想在你spider中模拟POST表单，发送几个key-value字段，你可以从你的spider中返回一个 `FormRequest` 对象，就像这样：

	return [FormRequest(url="http://www.example.com/post/action",
	                    formdata={'name': 'John Doe', 'age': '27'},
	                    callback=self.after_post)]

## 使用FormRequest.from\_response()模拟用户登录(Using FormRequest.from_response() to simulate a user login) ##

通常网站通过 <font color=red>`<input type='hidden'>`</font> 元素提供了预填充的表单字段，例如会话相关数据或者身份验证令牌(为了登录)。当爬取时，你想要这些数据自动预填充，只需要覆盖用户名和密码这样的字段。你可以使用 `FormRequest.from_response()` 方法完成这个任务。这里有一个示例：

	import scrapy
	
	class LoginSpider(scrapy.Spider):
	    name = 'example.com'
	    start_urls = ['http://www.example.com/users/login.php']
	
	    def parse(self, response):
	        return scrapy.FormRequest.from_response(
	            response,
	            formdata={'username': 'john', 'password': 'secret'},
	            callback=self.after_login
	        )
	
	    def after_login(self, response):
	        # check login succeed before going on
	        if "authentication failed" in response.body:
	            self.logger.error("Login failed")
	            return
	
	        # continue scraping with authenticated session...


# Response对象(Response objects) #

<table><tr><td>
<font color=green>class</font> scrapy.http.Response(url[, status=200, headers=None, body=b'', flags=None, request=None])
</td></tr></table>

一个 `Response` 对象代表一个HTTP响应，通常被下载(通过下载器)，然后传递给Spiders处理。

**参数：**

  - **url**(string) - 这个响应的url
  - **status**(string) - 响应的HTTP状态码。默认为 <font color=red>`200`</font>
  - **headers**(dict) - 响应头。字典的值可以为字符串(单值)或者列表(多值)
  - **body**(bytes) - 响应体。获取解码后的文本你可以使用能解码的Response的子类(例如 `TextResponse`) 的 <font color=red>`response.text`</font> 方法。
  - **flags**(list) - 一个包含`Response.flags`初始值的列表。如果给定，这个列表会被浅拷贝。
  - **request**(`Request`对象) - `Response.request` 属性的初始值。代表是这个 `Request` 生成这个响应。

**url**：</br>
包含响应url的字符串。只读，改变url使用 `replace()`方法。

**status**：</br>
HTTP响应码的整数。例如：<font color=red>`200`</font>、<font color=red>`404`</font>

**headers**：</br>
一个包含响应头的类字典对象。通过 `get()`方法取值返回特定名字的第一个值，`getlist()`方法返回特定名字所有值组成的列表。例如，这样调用将会返回响应头中所有的cookies：

	response.headers.getlist('Set-Cookie')

**body**：</br>
响应体。记住 `Response.body` 总是字节对象(bytes object)。如果你想要unicode，使用`TextResponse.text`(仅在 `TextResponse` 和它的子类中可用)

**request**：</br>
生成相应的`Request`对象。这个属性是在Scrapy引擎中分配，然后response和request将会传递给下载中间件。特别地，这意味着：</br>
  
  - HTTP重定向将会造成原始request(请求重定向的url)会分配给重定向后的response(与最终重定向后的url一起)。
  - `Response.request.url` 不总是等于 `Response.url`
  - 这个属性只在spider代码和爬虫中间件中可用，下载中间件中(虽然这里你有其他方式使用Request)和`response_downloaded`信号的发送者不可用。

**meta**：</br>
`Response.meta` = `Response.request.meta`，是获取 `Request.meta`的便捷方式。

不像 `Response.request` 属性，`Response.meta` 属性沿着重定向和重试传播，因此你将得到从spider中发送而来原始的 `Request.meta`。

**flags**：</br>
一个包含响应的flags的列表。Flags是用于标志响应的标签。例如：*'cached'*, *'redirected'*等等，它们显示代表Reponse的字符串(__str__方法），用作引擎的日志记录。

**copy**：</br>
通过Response复制一个新的Response对象并返回。

**replace([url, status, headers, body, request, flags, cls])**：</br>
返回具有相同成员的Response对象，除非这些成员通过各自特定的关键字参数指定了新的值。属性 `Reponse.meta` 默认情况下会被复制。

**urljoin**：</br>
通过结合Reponse的`url`，把一个可能的相对url构造成绝对url。封装了urlparse.urljoin，这仅仅是调用这个方法的别名而已：

	urlparse.urljoin(response.url, url)

**follow(url, callback=None, method='GET', headers=None, body=None, cookies=None, meta=None, encoding='utf-8', priority=0, dont_filter=False, errback=None)**：</br>
返回一个`Request`示例。它接受的参数与 <font color=red>`Request.__init__()`</font>方法相同，但是 <font color=red>`url`</font> 可以为相对url或者 <font color=red>scrapy.link.Link</font>对象，而不局限于绝对url。

`TextResponse` 通过了 `follow()` 方法除了支持相对/绝对/Link对象之外，还支持选择器。

# Reponse子类(Response subclass) #

这里列出了内置可用的Response子类。也可以自己定义子类。

## TextResponse对象(TextResponse objects) ##

<table><tr><td>
<font color=green>class</font> scrapy.http.TextResponse(url[,  encoding, status=200, headers=None, body=b'', flags=None, request=None])
</td></tr></table>

`TextResponse` 相对于 `Response` 类增加了解码的功能。`Response` 只能用在二进制数据，例如图片，声音，或者其他多媒体文件。

`TextResponse` 除了`Response`之外支持一个新的构造参数。其余的功能和 `Response` 一致。

**参数**：

  - **encoding**(string) - 一个包含此响应编码的字符串。如果你用一个unicode响应体创造了`TextResponse`对象，它将会使用这个编码方式编码(记住body属性依然是一个字符串)。如果 <font color=red>`encoding`</font> 是 <font color=red>`None`</font>(默认值)，encoding将会在想响应头和响应体中寻找。

`TextRespone` 还支持下面额外的属性：

**text**：</br>
unicode的响应体。</br>
和 <font color=red>`response.body.decode(response.encoding)`</font> 一样，但是第一次调用后，得到的结果会缓存，因此你多次调用 <font color=red>`response.text`</font> 不需要额外的开销。

<font color=orange>
注意：</br>
<font color=red>`unicode(response.body)`</font>不是将响应体转换为unicode的正确方法：您将使用系统默认编码（通常为ascii）而不是响应编码。
</font>

**encoding**：
reponse的编码方式。按照顺序通过以下机制解决获得encoding的值：

  1. 通过构造函数中传递*encoding*参数。
  2. 响应头中Content-Type声明的编码方式。如果值是无效的(例如为止)，将会被忽略，然后尝试下一个机制。
  3. 响应体中声明的编码方式。`TextResponse`没有提供特定的功能从响应体中找到编码方式。然而，`HtmlResponse`和`XmlResponse`类做到了。
  4. 通过响应体推倒出编码方式。这是一个更加脆弱的方法，但是最后也会尝试。

**selector**：</br>
以response为目标的`Selector`对象。这个对象在第一次被调用时才会被实例化。

`TextResponse`除了 `Response`的标准方法外还支持下面的方法：

**xpath(query)**：
<font color=red>`TextResponse.selector.xpath(query)`</font>的快捷方式：

	response.xpath('//p')

**css(query)**：
<font color=red>`TextResponse.selector.css(query)`</font>的快捷方式：

	response.css('p')

**follow(url, callback=None, method='GET', headers=None, body=None, cookies=None, meta=None, encoding=None, priority=0, dont_filter=False, errback=None)**：</br>
返回一个 `Request` 实例。它接受与 <font color=red>`Request.__init__()`</font> 方法相同的参数，但是 <font color=red>`url`</font> 不但可以为绝对url，而且可以为：

  - 相对url
  - scrapy.link.Link对象(例如，link extrator结果)
  - Selector(不是 SelectorList) - 例如：<font color=red>`response.css('a::attr(href)')[0]`</font>或者<font color=red>`response.xpath('//img/@src')[0]`</font> 
  - <font color=red>`<a>`</font> 或者<font color=red>`<link>`</font> 元素的Selector对象，例如 <font color=red>`response.css('a.my_link')[0]`</font>

**body\_as\_unicode()**：</br>
与`text`一样，但是作为一个方法使用。保留此方法是为了向后兼容；更喜欢：<font color=red>`response.text`</font>

## HtmlResponse对象(HtmlResponse objects) ##

<table><tr><td>
<font color=green>class</font> scrapy.http.HtmlResponse(url[,  encoding, status=200, headers=None, body=b'', flags=None, request=None])
</td></tr></table>

`HtmlResponse`是`TextResponse`的子类，增加了支持在HTML meta http-equiv属性中自动寻找编码方式。

## XmlResponse对象(XmlResponse objects) ##

<table><tr><td>
<font color=green>class</font> scrapy.http.XmlResponse(url[,  encoding, status=200, headers=None, body=b'', flags=None, request=None])
</td></tr></table>

`HtmlResponse`是`TextResponse`的子类，增加了支持在XML声明行中自动寻找编码方式。