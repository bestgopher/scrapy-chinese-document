#Logging#

<font color=orange>
NOTE:</br>
`scrapy.log` 已经弃用它的方法，支持显式调用Python标准库logging。继续阅读学习关于新的日志系统更多内容。
</font>

Scrapy使用Python内建日志系统(Python's bulitin logging system)来处理事件日志。我们将提供一些简单的示例带你开始，但是更多高级的使用示例我们强烈建议你通读它的文档。

Logging开箱即用，你可以在Scrapy settings中配置日志的级别。

当运行命令时，Scrapy调用 `scrapy.utils.log.config_logging()` 设置一些合理的默认值，处理Logging settings中的设置，因此如果你通过脚本运行Scrapy，推荐手动调用日志。

##Log levels##

Python的内建logging定义了5个不同的级别来指明给定日志信息的严重性。如下所示：

  1. <font color=red>`logging.CRITICA`</font> - 严重错误(严重性程度最高)
  2. <font color=red>`logging.ERROR`</font> - 一般错误
  3. <font color=red>`logging.WARNING`</font> - 警告信息
  4. <font color=red>`logging.INFO`</font> - 一般信息
  5. <font color=red>`logging.DEBUG`</font> - 调试信息(严重性程度最低)

##怎么记录信息(How to log messages)##

以下是怎么使用 <font color=red>`logging.WARNING`</font> 级别记录信息的快速示例：

	import logging
	logging.warning("This is a warning")

5个标准日志级别都有各自的快捷方式发出日志信息，也有一个普通的 <font color=red>`logging.log`</font> 方法，这个方法接受一个给定的级别作为参数。如果需要的话，上一个例子可以重写为这样：

	import logging
	logging.log(logging.WARNING, "This is a warning")

除此之外，你可以创建不同的'loggers'来封装信息。(例如，常见的做法是为每个模板创建不同的loggers)。这些loggers能够单独配置，并允许分层构造。

先前的例子在后面使用的是root logger，这是一个最高等级的logger，所有的信息都会传递过来(除非另外特定的)。使用 <font color=red>`logging`</font> 仅仅是一个获得root logger的快捷方式而已，因此下面的示例与上一个例子一样：

	import logging
	logger = logging.getLogger()
	logger.warning("This is a warning")

你可以使用不同的logger，只需要通过传递它的名字到 <font color=red>`logging.getLogger`</font>函数中：

	import logging
	logger = logging.getLogger('mycustomlogger')
	logger.warning("This is a warning")

最后，你可以确保让每个模块拥有一个自定义的logger，你可以通过 <font color=red>`__name__`</font> 变量填充当前模块的路径获得。
	
	import logging
	logger = logging.getLogger(__name__)
	logger.warning("This is a warning")

##在Spider中添加log(Logging from Spiders)##

Scrapy在每个spider实例内部提供一个 `logger`，可以这样使用它：

	import scrapy
	
	class MySpider(scrapy.Spider):
	
	    name = 'myspider'
	    start_urls = ['https://scrapinghub.com']
	
	    def parse(self, response):
	        self.logger.info('Parse function called on %s', response.url)

这个logger通过使用spider的名字创建的，但是你也可以使用任何Python自定义的logger。例如：

	import logging
	import scrapy
	
	logger = logging.getLogger('mycustomlogger')
	
	class MySpider(scrapy.Spider):
	
	    name = 'myspider'
	    start_urls = ['https://scrapinghub.com']
	
	    def parse(self, response):
	        logger.info('Parse function called on %s', response.url)

##(配置log)Logging configuration##

Loggers本身不会管理那些显示的消息怎么发送的。对于这个任务，不同的'处理器(handlers)'依附在logger实例中，它们将重定向这些信息到合适的目的地，例如标准输出，文件，邮箱等。

默认情况下，Scrapy为root loggers配置了一个处理器，基于下面的设置。

##log设置(Logging settings)##

这些设置用来配置logging：

   - `LOG_FILE`
   - `LOG_ENABLED`
   - `LOG_ENCODING`
   - `LOG_LEVEL`
   - `LOG_FORMAT`
   - `LOG_DATEFORMAT`
   - `LOG_STDOUT`
   - `LOG_SHORT_NAMES`

前几个设置定义了日志的目的地。如果 `LOG_FILE` 设置了，通过root logger发送的消息将会使用`LOG_ENCODING`的编码方式被重定向到`LOG_FILE`文件中。如果没有设置 `LOG_ENABLED` 为 <font color=red>`True`</font>，log信息将会在标准错误输出(standard error)上显示。最后，如果 `LOG_ENABLED` 为 <font color=red>`False`</font>，将不会有任何明显的日志输出。

`LOG_LEVEL` 决定了显示的最低严重性等级，更低严重性的消息将会被过滤掉。可用的级别列在 Log levels中。

`LOG_FORMAT`和 `LOG_DATEFORMAT` 指定了格式的字符串，用作信息的布局。这些字符串包含了一些占位符，这些占位符分别列举在https://docs.python.org/2/library/logging.html#logrecord-attributes和https://docs.python.org/2/library/datetime.html#strftime-and-strptime-behavior。

如果 `LOG_SHORT_NAMES` 设置了，打印log时将不会显示scrapy组件。
默认没有设置，因此log输出时包含了scrapy的组件的。

##命令行选项(Command-line options）##

这里有一些适用于**所有命令**的命令行选项，你可以使用它们重写一些Scrapy关于log的设置。
 
  - <font color=red>`--logfile FILE`</font>：重写 `LOG_FILE`
  - <font color=red>`--loglevel/-L LEVEL`</font>：重写 `LOG_LEVEL`
  - <font color=red>`--nolog`</font>：设置 `LOG_ENABLED`为<font color=red>`False`</font>

##高级定制(Advanced customization)##

因为Scrapy使用的是标准库logging模块，你可以使用所有标准库logging中所有的特性来定制scrapy的logging。

例如，你爬取的网站返回许多HTTP404和500的响应，并且你想要隐藏像下面的这些信息：

	2016-12-16 22:00:06 [scrapy.spidermiddlewares.httperror] INFO: Ignoring
	response <500 http://quotes.toscrape.com/page/1-34/>: HTTP status code
	is not handled or not allowed

首先要注意的就是logger的名字 - 方括号中：<font color=red>`[scrapy.spidermiddlewares.httperror]`</font>。如果你得到的仅仅只有<font color=red>`[scrapy]`</font>，很可能 `LOG_SHORT_NAMES`设置为 <font color=red>`True`</font>; 把`LOG_SHORT_NAMES`设置为 <font color=red>`False`</font>，然后重新运行爬虫。

接下来，我们可以看见小心是INFO级别的。为了隐藏我们可以为<font color=red>`[scrapy.spidermiddlewares.httperror]`</font>设置级别高于INFO；INFO的下一个级别是WARNING。我们可以在spider的 <font color=red>`__init__`</font>方法中完成设置。

	import logging
	import scrapy
	
	
	class MySpider(scrapy.Spider):
	    # ...
	    def __init__(self, *args, **kwargs):
	        logger = logging.getLogger('scrapy.spidermiddlewares.httperror')
	        logger.setLevel(logging.WARNING)
	        super().__init__(*args, **kwargs)


如果你再一次运行spider，从 <font color=red>`[scrapy.spidermiddlewares.httperror]`</font>logger发出的INFO消息将会被隐藏。

#scrapy.utils.log module#

<table><tr><td>
scrapy.utils.log.configure_logging(settings=None, install_root_handler=True)
</td></tr></table>

为Scrapy初始化logging。

**参数**：
  
  - **settings**(字典，`setting`对象或者<font color=red>None</font>) - settings用来为root logger创建和配置一个处理器(handler)，默认为None。
  - **install\_root\_handler**(bool) - 是否安装root logging处理器(默认：True)


这个函数的作用：

  - 通过Python标准库logging路由警告和twisted logging
  - 分别为Scrapy和Twisted loggers声明DEBUG和ERROR等级。
  - 如果`LOG_STDOUT`设置为True，路由到标准输出到log。

当 <font color=red>`install_root_handler`</font> 设置为True(默认情况下)，这个函数根据给定的settings为root logger创建一个处理器。你可以通过<font color=red>`settings`</font>参数重写默认选项。当<font color=red>`settings`</font>是None或者空，使用默认值。

<font color=red>`configure_logging`</font>在使用Scrapy命令时自动调用，但是当通过自定义脚本运行时需要显式地调用。这种情况下，虽然不需要但是也这样建议。

如果你计划自定义处理器，仍然推荐你调用这个函数，设置 <font color=red>`install_root_handler=False`</font>。记住这种情况下默认将不会有任何log输出。

手动配置日志输出，你可以使用 <font color=red>`logging.basicConfig()`</font>设置基本root handler。下面是一个示例，举例怎么重定向 <font color=red>`INFO`</font>或更高基本信息到一个文件中。

	import logging
	from scrapy.utils.log import configure_logging
	
	configure_logging(install_root_handler=False)
	logging.basicConfig(
	    filename='log.txt',
	    format='%(levelname)s: %(message)s',
	    level=logging.INFO
	)