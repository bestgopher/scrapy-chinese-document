#异常(Exceptions)#

#内置异常参考手册(Built-in Exceptions reference)#

下面是Scrapy提供的异常及其用法。

##DropItem##

<table><tr><td>
<font color=green>exception</font> scrapy.exceptions.DropItem
</td></tr></table>

该异常由item pipeline抛出，用于停止处理item。详细内容请参考 Item Pipeline 。

##CloseSpider##

<table><tr><td>
<font color=green>exception</font> scrapy.exceptions.CloseSpider(reason='cancelled')
</td></tr></table>

**参数**:	reason (str) – 关闭的原因

样例:

	def parse_page(self, response):
	    if 'Bandwidth exceeded' in response.body:
	        raise CloseSpider('bandwidth_exceeded')

##DontCloseSpider##

<table><tr><td>
<font color=green>exception</font> scrapy.exceptions.DontCloseSpider
</td></tr></table>

这个异常可以在 `spider_idle` 信号处理器中抛出，阻止spider被关闭。

##IgnoreRequest##

<table><tr><td>
<font color=green>exception</font> scrapy.exceptions.IgnoreRequest
</td></tr></table>

该异常由调度器(Scheduler)或其他下载中间件抛出，声明忽略该request。

##NotConfigured##

<table><tr><td>
<font color=green>exception</font> scrapy.exceptions.NotConfigured
</td></tr></table>

该异常由某些组件抛出，声明其仍然保持关闭。这些组件包括:

  - Extensions
  - Item pipelines
  - Downloader middlwares
  - Spider middlewares

该异常必须由组件的 <font color=red>`__init__`</font> 抛出。

##NotSupported##

<table><tr><td>
<font color=green>exception</font>  scrapy.exceptions.NotSupported
</td></tr></table>

该异常声明一个不支持的特性。