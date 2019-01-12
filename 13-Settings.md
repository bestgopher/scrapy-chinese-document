#Settings#

Scrapy设定(settings)提供了定制Scrapy组件的方法。您可以控制包括核心(core)，插件(extension)，pipeline及spider组件。

设定为代码提供了提取以key-value映射的配置值的的全局命名空间(namespace)。 设定可以通过下面介绍的多种机制进行设置。

设定(settings)同时也是选择当前激活的Scrapy项目的方法(如果您有多个的话)。

内置设定列表请参考 内置设定参考手册 。

#指定设定(Designating the settings)#

当您使用Scrapy时，您需要声明您所使用的设定。这可以通过使用环境变量: <font color=red>`SCRAPY_SETTINGS_MODULE`</font> 来完成。

<font color=red>`SCRAPY_SETTINGS_MODULE`</font> 必须以Python路径语法编写, 如 <font color=red>`myproject.settings`</font> 。 注意，设定模块应该在 Python `import search path` 中。

#获取设定值(Populating the settings)#

设定可以通过多种方式设置，每个方式具有不同的优先级。 下面以优先级降序的方式给出方式列表:

  1. 命令行选项(Command line Options)(最高优先级)
  2. 每个spider的设定
  3. 项目设定模块(Project settings module)
  4. 命令默认设定模块(Default settings per-command)
  5. 全局默认设定(Default global settings) (最低优先级)

这些设定(settings)由scrapy内部很好的进行了处理，不过您仍可以使用API调用来手动处理。 详情请参考 设置`(Settings API）`。

这些机制将在下面详细介绍。

##1. 命令行选项(Command line options)##

命令行传入的参数具有最高的优先级。 您可以使用command line 选项 -<font color=red>`s`</font> (或 <font color=red>`--set`</font>) 来覆盖一个(或更多)选项。

样例:

	scrapy crawl myspider -s LOG_FILE=scrapy.log

##2. 为每个spider设置(Settings per-spider)##

Spider可以定义自己的设置，这些设置的优先级最高，会覆盖项目的设置。它们可以通过 `custom_settings` 属性定义：

	class MySpider(scrapy.Spider):
	    name = 'myspider'
	
	    custom_settings = {
	        'SOME_SETTING': 'some value',
	    }

##3. 项目设定模块(Project settings module)##

项目设定模块是您Scrapy项目的标准配置文件。 其是获取大多数设定的方法。对于一个标准的Scrapy项目，这就意味着你添加或者修改这些设置在为你项目创建的 <font color=red>`settings.py`</font> 文件中。

##4. 命令默认设定(Default settings per-command)##

每个 `Scrapy tool` 命令拥有其默认设定，并覆盖了全局默认的设定。 这些设定在命令的类的 <font color=red>`default_settings`</font> 属性中指定。

##5. 默认全局设置(Default global settings)##

全局默认设定存储在  <font color=red>`scrapy.settings.default_setting`s</font> 模块， 并在 内置设定参考手册 部分有所记录。

#如何访问设定(How to access settings)#

在spider中，设置key通过 <font color=red>`self.settings`</font> 获得。

	class MySpider(scrapy.Spider):
	    name = 'myspider'
	    start_urls = ['http://example.com']
	
	    def parse(self, response):
	        print("Existing settings: %s" % self.settings.attributes.keys())

<font color=orange>
NOTE:</br>
<font color=red>`settings`</font> 属性是在spider实例化之后在基础Spider类中设置的。如果你想在实例化之前使用设置(例如在你spider的`__init__()` 方法中)，你需要重写 `from_crawler()` 方法
</font>

设置可以通过Crawler的 `scrapy.crawler.Crawler.settings` 属性获得，Crawler会传递给扩展(extensions)，中间件(middlewares)，项目管道(item pipeline)的 `from_crawler` 方法：
	
	class MyExtension(object):
	    def __init__(self, log_is_enabled=False):
	        if log_is_enabled:
	            print("log is enabled!")
	
	    @classmethod
	    def from_crawler(cls, crawler):
	        settings = crawler.settings
	        return cls(settings.getbool('LOG_ENABLED'))

settings对象可以像字典一样使用(例如: <font color=red>`settings['LOG_ENABLED']`</font>)，但是通常优先使用 `Settings` API提供的方法来按格式获取你需要的setting，从而避免类型错误(type errors)。

#设定名字的命名规则(Rationale for setting names)#

设定的名字以要配置的组件作为前缀。 例如，一个robots.txt插件的合适设定应该为 <font color=red>`ROBOTSTXT_ENABLED`</font>, <font color=red>`ROBOTSTXT_OBEY`</font>, <font color=red>`ROBOTSTXT_CACHEDIR`</font> 等等。