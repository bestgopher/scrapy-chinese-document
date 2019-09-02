# Link Extractors #

Link Extractors 是用于从网页(`scrapy.http.Response` 对象 )中抽取会被follow的链接的对象。

Scrapy中有个 <font color=red>`scrapy.linkextractors.LinkExtractor`</font>类可用，不过您也可以通过实现一个简单的接口来创建您自己的Link Extractor，满足需求。

每个LinkExtractor有唯一的公共方法是 <font color=red>`extract_links`</font> ，其接收 一个 `Response` 对象， 并返回 `scrapy.link.Link` 对象｡ Link Extractors只实例化一次，其 `extract_links` 方法会根据不同的response被调用多次来提取链接｡

Link Extractors在 `CrawlSpider` 类(在Scrapy可用)中使用, 通过一套规则,但你也可以用它在你的Spider中,即使你不是从 `CrawlSpider` 继承的子类, 因为它的目的很简单: 提取链接｡

# 内置Link Extractor 参考(Built-in link extractors reference) #

Scrapy 自带的Link Extractors类在 `scrapy.contrib.linkextractors` 模块提供｡

默认的link extractor是 <font color=red>`LinkExtractor`</font>，它和 `LxmlLinkExtractor` 一样的。

	from scrapy.linkextractors import LinkExtractor

以前的版本有其他link extractor类，但如今被弃用了。

## LxmlLinkExtractor ##

<table>
<tr>
<td>
<font color=green>class</font> scrapy.linkextractors.lxmlhtml.LxmlLinkExtractor(allow=(), deny=(), allow_domains=(), deny_domains=(), deny_extensions=None, restrict_xpaths=(), restrict_css=(), tags=('a', 'area'), attrs=('href', ), canonicalize=False, unique=True, process_value=None, strip=True)
</td>
</tr>
</table>

`LxmlLinkExtrator` 是推荐使用的link extractor，有着便利的过滤选项。它是使用`lxml`强大的`HTMLParse`实现的。

**参数**：

  - **allow**(一个正则或者正则列表) -  单个正则表达式(正则列表)，(绝对)urls必须完全匹配才能提取。如果没有给定(或者为空)，它将会匹配所有的链接。
  - **deny**(一个正则或者正则列表) - 单个正则表达式(正则列表)，(绝对)urls必须完全匹配才能被排除(不是提取)。它优先于<font color=red>`allow`</font>参数。如果没有给定(或者为空)，它将不会提取任何链接。
  - **allow_domains**(字符串或者列表) - 单个字符串或者字符串列表，包含允许提取的链接的域名。
  - **deny_domains**(字符串或者列表) - 单个字符串或者字符串列表，包含不允许允许提取的链接的域名。
  - **deny_extensions**(列表) - 一个或一列表的值，包含提取链接时应该忽略的链接。如果没有给定，默认为 scrapy.linkextractors包中定义的 <font color=red>`IGNORED_EXTENSIONS`</font> 列表。
  - **restrict_xpaths**(字符串或者列表) - 一个字符串或者字符串列表，定义response的某个区域，链接从这个区域中提取。如果给定了，链接仅从由Xpath选择的文本区域中扫描出来。看下面的例子。
  - **restrict_css**(字符或者列表) - 一个css选择器(或者css选择器的列表)，定义response的某个区域，链接从这个区域中提取。有着与<font color=red>`restrict_xpaths`</font>一样的行为。
  - **tags**(字符串或者列表) - 一个html标签或着列表。默认为<font color=red>`('a', 'area')`</font>。
  - **attrs**(列表) - 一个属性或者列表，查找链接时要考虑的(只有这些标签在<font color=red>`tags`</font>参数中指定)。默认为<font color=red>`('href',)`</font>
  - **canonicalize**(布尔值) - 规范化每个提取的url(使用`w3lib.url.canonicalize_url`)。默认为<font color=red>`False`</font>。规范化url是为了查重；它能在服务器端明显地改变URL，因此原始url得到的响应可能与规范化后的响应不同。如果你想使用 `LinkExtrator` follow链接的话，最好使用默认的 <font color=red>`canonicalize=False`</font>。
  - **unique**(布尔值) - 是否对链接进行重复过滤操作。
  - **process_value**(可调用的) - 函数，接受从标签和属性中提取出来的每个值，改变这些值，返回新的值，返回<font color=red>`None`</font>完全忽略这个值。如果没有给定，<font color=red>`process_value`</font>默认为<font color=red>`lambda x: x`</font>。</br>例如，从这个代码中提取链接:</br>		
     	`a href="javascript:goToPage('../other/page.html'); return false">Link text</a>`
  你可以在 <font color=red>`process_value`</font>中使用下面的函数：

		def process_value(value):
		    m = re.search("javascript:goToPage\('(.*?)'", value)
		    if m:
		        return m.group(1)
  - **strip**(布尔值) - 是否从提取的属性中去掉空格。根据HTML标准，<font color=red>`<a>`</font>、<font color=red>`<area>`</font>中的<font color=red>`href`</font>，<font color=red>`<iframe>`</font>、<font color=red>`<img>`</font>中的<font color=red>`src`</font>，等等，标签属性中前后的空格必须去掉。因此默认情况下 `LinkExtract`会去掉空字符。设置 <font color=red>`strip=False`</font>关掉它(例如你从元素或者属性中提取的url运行前后有空白字符)。
     	

