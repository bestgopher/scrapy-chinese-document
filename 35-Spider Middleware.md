#Spider Middleware#

Spider中间件是介入到Scrapy的spider处理机制的钩子框架，您可以添加代码来处理发送给 Spiders 的response及spider产生的item和request。

##激活spider中间件(Activating a spider middleware)##

要启用spider中间件，您可以将其加入到 SPIDER_MIDDLEWARES 设置中。 该设置是一个字典，键位中间件的路径，值为中间件的顺序(order)。

样例:

	SPIDER_MIDDLEWARES = {
	    'myproject.middlewares.CustomSpiderMiddleware': 543,
	}

`SPIDER_MIDDLEWARES` 设置会与Scrapy定义的 `SPIDER_MIDDLEWARES_BASE` 设置合并(但不是覆盖)， 而后根据顺序(order)进行排序，最后得到启用中间件的有序列表: 第一个中间件是最靠近引擎的，最后一个中间件是最靠近spider的。换句话说，每个中间件的`process_spider_input` 方法将会按照中间件的递增顺序(100，200，300)调用，每个中间件的`process_spider_ouput`方法将会按降序调用。

关于如何分配中间件的顺序请查看 `SPIDER_MIDDLEWARES_BASE` 设置，而后根据您想要放置中间件的位置选择一个值。 由于每个中间件执行不同的动作，您的中间件可能会依赖于之前(或者之后)执行的中间件，因此顺序是很重要的。

如果您想禁止内置的(在 SPIDER_MIDDLEWARES_BASE 中设置并默认启用的)中间件， 您必须在项目的 SPIDER_MIDDLEWARES 设置中定义该中间件，并将其值赋为 None 。 例如，如果您想要关闭off-site中间件:

	SPIDER_MIDDLEWARES = {
	    'myproject.middlewares.CustomSpiderMiddleware': 543,
	    'scrapy.spidermiddlewares.offsite.OffsiteMiddleware': None,
	}

最后，请注意，有些中间件需要通过特定的设置来启用。更多内容请查看相关中间件文档。

##编写您自己的spider中间件(Writing your own spider middleware)##

每个中间件组件是一个定义了以下一个或多个方法的Python类：

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spidermiddlewares.SpiderMiddleware
</td></tr></table>

**process\_spider\_input(response, spider)**

当response通过spider中间件时，该方法被调用，处理该response。

`process_spider_input()` 应该返回 <font color=red>`None`</font> 或者抛出一个异常。

如果其返回 <font color=red>`None`</font> ，Scrapy将会继续处理该response，调用所有其他的中间件直到spider处理该response。

如果其抛出一个异常(exception)，Scrapy将不会调用任何其他中间件的 `process_spider_input()` 方法，并调用request的errback。 errback的输出将会以另一个方向被重新输入到中间件链中，使用 `process_spider_output()` 方法来处理，当其抛出异常时则带调用 `process_spider_exception()` 。

&nbsp;&nbsp;&nbsp;**参数**：
  
 - **response**(`Response`对象 - 被处理的 response
 - **spider**(`Spider`对象 - 该response对应的spider


**process\_spider\_output(response, result, spider)**

当Spider处理response返回result时，该方法被调用。

`process_spider_output()` 必须返回包含 `Request` 或 `Item` 对象的可迭代对象(iterable)。

&nbsp;&nbsp;&nbsp;**参数**：
  
 - **response**(`Response`对象) - 生成该输出的response
 - **result**(包含 `Request` 或 `Item` 对象的可迭代对象(iterable) - spider返回的result
 - **spider**(`Spider` 对象) - 其结果被处理的spider


**process\_spider_exception(response, exception, spider)**

当spider或(其他spider中间件的) `process_spider_input()` 跑出异常时， 该方法被调用。

process_spider_exception() 必须要么返回 <font color=red>`None`</font> ， 要么返回一个包含 `Response` 或 `Item` 对象的可迭代对象(iterable)。

如果其返回 <font color=red>`None`</font> ，Scrapy将继续处理该异常，调用中间件链中的其他中间件的 `process_spider_exception() `方法，直到所有中间件都被调用，该异常到达引擎(异常将被记录并被忽略)。

如果其返回一个可迭代对象，则中间件链的 `process_spider_output()` 方法被调用， 其他的 `process_spider_exception()` 将不会被调用。

&nbsp;&nbsp;&nbsp;**参数**：

 - **response** (`Response` 对象) – 异常被抛出时被处理的response
 - **exception** (`Exception` 对象) – 被抛出的异常
 - **spider** (`Spider` 对象) – 抛出该异常的spider

**process\_start\_requests(start_requests, spider)**

该方法以spider 启动的request为参数被调用，执行的过程类似于 `process_spider_output()` ，只不过其没有相关联的response并且必须返回request(不是item)。

其接受一个可迭代的对象(<font color=red>`start_requests`</font> 参数)且必须返回另一个包含 `Request` 对象的可迭代对象。

<font color=orange>
NOTE：</br>
当在您的spider中间件实现该方法时， 您必须返回一个可迭代对象(类似于参数start_requests)且不要遍历所有的 <font color=red>`start_requests`</font>。 该迭代器会很大(甚至是无限)，进而导致内存溢出。 Scrapy引擎在其具有能力处理start request时将会拉起request， 因此start request迭代器会变得无限，而由其他参数来停止spider( 例如时间限制或者item/page记数)。

</font>

&nbsp;&nbsp;&nbsp;**参数**：

  - **start_requests** (包含 `Request` 的可迭代对象) -  start requests
  - **spider**( `Spider` 对象) – start requests所属的spider


**from_crawler(cls, crawler)**

如果存在，这个类方法被调用从`Crawler`中创建一个中间件实例。它必须返回一个新的中间件实例。Crawler对象提供访问所有Scrapy核心组件，例如settings和signals；这是middleware去访问这些组件和使中间件的功能与Scrapy关联的一种方式。

&nbsp;&nbsp;&nbsp;**参数**：

  - **crawler**('Crawler'对象) - 使用中间件的crawler。

##内置spider中间件参考手册(Built-in spider middleware reference)##

本页面介绍了Scrapy自带的所有spider中间件。关于如何使用及编写您自己的中间件，请参考 spider middleware usage guide.

关于默认启用的中间件列表(及其顺序)请参考 `SPIDER_MIDDLEWARES_BASE` 设置。

##DepthMiddleware##

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spidermiddlewares.depth.DepthMiddleware
</td></tr></table>

DepthMiddleware是一个用于追踪每个Request在被爬取的网站的深度的中间件。设置 request.meta['depth'] = 0 这个中间件就会工作，
每当没有值预先设置时(通常为第一个请求)，以1递增。

可以用来限制最大的爬取深度，通过深度控制请求的优先级。

`DepthMiddleware` 可以通过下列设置进行配置(更多内容请参考设置文档)：

  - **DEPTH_LIMIT** - 爬取所允许的最大深度，如果为0，则没有限制。
  - **DEPTH_STATS** - 是否收集爬取深度状态。
  - **DEPTH_PRIORITY** - 是否根据其深度对requet安排优先级


##HttpErrorMiddleware##

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spidermiddlewares.httperror.HttpErrorMiddleware
</td></tr></table>

过滤出所有失败(错误)的HTTP response，因此spider不需要处理这些request。 处理这些request意味着消耗更多资源，并且使得spider逻辑更为复杂。

根据 HTTP标准 ，返回值为200-300之间的值为成功的response。

如果您想处理在这个范围之外的response，您可以通过 spider的 <font color=red>`handle_httpstatus_list`</font> 属性或` HTTPERROR_ALLOWED_CODES` 设置来指定spider能处理的response返回值。

例如，如果您想要处理返回值为404的response您可以这么做：

	class MySpider(CrawlSpider):
	    handle_httpstatus_list = [404]

`Request.meta` 中的 <font color=red>`handle_httpstatus_list`</font> 键也可以用来指定每个request所允许的response code。如果你允许所有的状态码都能被spider处理，你也可以设置meta key <font color=red>`handle_httpstatus_all`</font> 为 <font color=red>`True`</font>。

不过请记住，除非您知道您在做什么，否则处理非200返回一般来说是个糟糕的决定。

###HttpErrorMiddleware settings###

#####HTTPERROR_ALLOWED\_CODES#####

默认：<font color=red>`[]`</font>

忽略该列表中所有非200状态码的response。

#####HTTPERROR_ALLOW\_ALL#####

默认：<font color=red>`False`</font>

忽略所有response，不管其状态值。

##OffsiteMiddleware##

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.contrib.spidermiddleware.offsite.OffsiteMiddleware
</td></tr></table>

过滤出所有URL不由该spider负责的Request。

该中间件过滤出所有主机名不在spider属性 `allowed_domains` 的request。所有列表中的子域名也能被允许使用，例如<font color=red>`www.example.org`</font> 允许<font color=red>`bob.www.example.org`</font>，但是不允许 <font color=red>`www2.example.org`</font> 或者 <font color=red>`example.org`</font>

当spide返回一个主机名不属于该spider的request时， 该中间件将会做一个类似于下面的记录：

	DEBUG: Filtered offsite request to 'www.othersite.com': <GET http://www.othersite.com/some/page.html>


为了避免记录太多无用信息，其将对每个新发现的网站记录一次。因此，例如， 如果过滤出另一个 <font color=red>`www.othersite.com`</font> 请求，将不会有新的记录。 但如果过滤出 <font color=red>`othersite.com`</font>请求，仍然会有记录信息(仅针对第一次)。

如果spider没有定义 `allowed_domains` 属性，或该属性为空， 则offsite 中间件将会允许所有request。

如果request设置了 `dont_filter` 属性， 即使该request的网站不在允许列表里，则offsite中间件将会允许该request

##RefererMiddleware##

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spidermiddlewares.referer.RefererMiddleware
</td></tr></table>

根据生成Request的Response的URL来设置Request <font color=red>`Referer`</font> 字段。

###RefererMiddleware settings###

#####REFERER_ENABLED#####

默认：<font color=red>`True`</font>

是否启用referer中间件。

#####REFERRER_POLICY#####

默认：<font color=red>`'scrapy.spidermiddlewares.referer.DefaultReferrerPolicy'`</font>

应用 [Referer策略](https://www.w3.org/TR/referrer-policy/) 生成Request请求头中的 'referer'。

<font color=orange>
NOTE：</br>
你可以为每个请求设置Referer Policy，使用指定的Request.meta键 <font color=red>`referrer_policy`</font>，接受 <font color=red>`REFERRER_POLICY`</font> 一样的值。
</font>

#####REFERRER_POLICY的可接受值(Acceptable values for REFERRER_POLICY)#####

  - 要么是 <font color=red>`scrapy.spidermiddlewares.referer.ReferrerPolicy`</font> 子类的路径(自定义)，要么是内置的中的一个。
  - 或者标准W3C定义的字符串值。
  - 或者特别的<font color=red>`"scrapy-default"`</font>

<table><tr><td><h5>String value</h5></td><td><h5>Class name (as a string)</h5></td></tr>
<tr><td><font color=red>"scrapy-default"</font> (default)</td><td>scrapy.spidermiddlewares.referer.DefaultReferrerPolicy</td></tr>
<tr><td>“no-referrer”</td><td>scrapy.spidermiddlewares.referer.NoReferrerPolicy</td></tr>
<tr><td>“no-referrer-when-downgrade”</td><td>scrapy.spidermiddlewares.referer.NoReferrerWhenDowngradePolicy</td></tr>
<tr><td>“same-origin”</td><td>	scrapy.spidermiddlewares.referer.SameOriginPolicy</td></tr>
<tr><td>“origin”</td><td>scrapy.spidermiddlewares.referer.OriginPolicy</td></tr>
<tr><td>“strict-origin”</td><td>scrapy.spidermiddlewares.referer.StrictOriginPolicy</td></tr>
<tr><td>“origin-when-cross-origin”</td><td>scrapy.spidermiddlewares.referer.OriginWhenCrossOriginPolicy</td></tr>
<tr><td>“strict-origin-when-cross-origin”</td><td>scrapy.spidermiddlewares.referer.StrictOriginWhenCrossOriginPolicy</td></tr>
<tr><td>“unsafe-url”</td><td>	scrapy.spidermiddlewares.referer.UnsafeUrlPolicy</td></tr>
</table>


<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spidermiddlewares.referer.DefaultReferrerPolicy
</td></tr></table>
 
'no-referrer-when-downgrade' 的变体，当父请求使用 <font color=red>`file://`</font> 或者 <font color=red>`s3://`</font> 协议时，将不会发送 'Referer'。

<font color=orange>
NOTE：</br>
Scrapy的默认referrer策略 - 与 'no-referrer-when-downgrade' ，这个W3C为浏览器推荐的值，相似。从 <font color=red>`http(s)://`</font> 到 <font color=red>`https://`</font> 的url， 将会发送一个非空的 'Referer' 头，即使域名不同。</br>'same-origin'可能是一个更好的选择，如果你想为跨域请求移除referer信息。
</font>

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spidermiddlewares.referer.NoReferrerPolicy
</td></tr></table>

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spidermiddlewares.referer.NoReferrerWhenDowngradePolicy
</td></tr></table>

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spidermiddlewares.referer.SameOriginPolicy
</td></tr></table>

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spidermiddlewares.referer.OriginPolicy
</td></tr></table>

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spidermiddlewares.referer.StrictOriginPolicy
</td></tr></table>

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spidermiddlewares.referer.OriginWhenCrossOriginPolicy
</td></tr></table>

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spidermiddlewares.referer.StrictOriginWhenCrossOriginPolicy
</td></tr></table>

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spidermiddlewares.referer.UnsafeUrlPolicy
</td></tr></table>

##UrlLengthMiddleware##

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.spidermiddlewares.urllength.UrlLengthMiddleware
</td></tr></table>

过滤出URL长度比URLLENGTH_LIMIT的request。

`UrlLengthMiddleware` 可以通过下列设置进行配置(更多内容请参考设置文档):

  - `URLLENGTH_LIMIT` - 允许爬取URL最长的长度.
