#Core API#
该节文档讲述Scrapy核心API，目标用户是开发Scrapy扩展(extensions)和中间件(middlewares)的开发人员。

##Crawler API##
Scrapy API的主要入口是 `Crawler` 的实例对象， 通过类方法 <font color=red>`from_crawler`</font> 将它传递给扩展(extensions)。 该对象提供对所有Scrapy核心组件的访问， 也是扩展访问Scrapy核心组件和挂载功能到Scrapy的唯一途径。

Extension Manager负责加载和跟踪已经安装的扩展， 它通过 EXTENSIONS 配置，包含一个所有可用扩展的字典， 字典的顺序跟你在 configure the downloader middlewares 配置的顺序一致。

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.crawler.Crawler(spidercls, settings)
</td></tr></table>

Crawler必须使用 `scrapy.spiders.Spider` 的子类的对象和`scrapy.settings.Settings` 的对象进行实例化

**settings**

crawler的配置管理器。

扩展(extensions)和中间件(middlewares)使用它用来访问Scrapy的配置。

关于Scrapy配置的介绍参考这里 Settings。

API参考 `Settings` 类。

**signals**

crawler的信号管理器。

扩展和中间件使用它将自己的功能挂载到Scrapy。

关于信号的介绍参考 信号(Signals)。

API参考 `SignalManager` 类。


**stats**

crawler的统计信息收集器。

扩展和中间件使用它记录操作的统计信息，或者访问由其他扩展收集的统计信息。

关于统计信息收集器的介绍参考 数据收集(Stats Collection)。

API参考类 `StatsCollector` 类。


**extensions**

扩展管理器，跟踪所有开启的扩展。

大多数扩展不需要访问该属性。

关于扩展和可用扩展列表器的介绍参考 扩展(Extensions)。


**engine**

执行引擎，协调crawler的核心逻辑，包括调度，下载和spider。

某些扩展可能需要访问Scrapy的引擎属性，以修改或者检查下载器和调度器的行为， 这是该API的高级使用，但还不稳定。

**spider**

当前爬取的spider。这是spider类的实例，在构造crawler时提供的，它在`crawl`方法给定了这个参数后被创建。

**crawl(\*args, \*\*kwargs)**

通过给定的args和kwargs参数实例化Spider类来启动爬虫，同时设置执行引擎。



<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.crawler.CrawlerRunner(settings=None)
</td></tr></table>

这是一个方便的类，在已设置的Twisted reactor里面跟踪，管理和运行crawlers。

这个CrawlerRunner对象必须通过 `Settings` 对象实例化。

这个类不需要使用，除非编写脚本手动处理爬取进程。

**crawl(crawler\_or_spidercls, \*args, \*\*kwargs)**

通过提供的参数运行crawler。

它将会调用给定的 Crawler类的 `crawl()` 方法，同时跟踪它，以便稍后停止。

如果 **crawler\_or_spidercls** 不是一个 `Crawler` 实例，这个方法将尝试使用这个参数创建一个spider类。

返回一个爬取结束时触发的deferred。

&nbsp;&nbsp;&nbsp;**参数**：

  - **crawler_or_spidercls**(`Crawler` 实例，`Spider` 子类，或者 字符串) - 已创建的crawler，或者spider的子类，或者项目中的spider的名字。
  - **args**(列表) - 实例化spider的参数
  - **kwargs**(字典) - 实例化spider的参数



**crawlers**

设置通过`crawl()` 方法开启，通过这个类管理的 `crawlers`。

**create_crawler(crawler\_or\_spidercls)**

返回一个`Crawler` 对象。

 - 如果 crawler\_or\_spidercls 是Crawler，按原样返回。
 - 如果 crawler\_or\_spidercls 是Spider子类，用这个子类构造新的Crawler。
 - 如果 crawler\_or\_spidercls 是字符串，这个函数在Scrapy项目中寻找名字为这个字符串的spider(使用spider加载器)，然后用这儿spider创建一个Crawler示例。

**join()**

返回当所有管理的 `crawler` 都完成了触发的deferred。

**stop()**

同时停止所有正在进行的爬行作业。

返回在它们全部结束时触发的deferred。


 <table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.crawler.CrawlerProcess(settings=None, install_root_handler=True)
</td></tr></table>
基类： `scrapy.crawler.CrawlerRunner`

一个同时在一个进程中运行多个scrapy crawler的类。

此类通过添加对启动一个Twisted reactor和处理关闭信号的支持来扩展 `CrawlerRunner`，例如键盘中断命令 Ctrl-C。它也配置了最高等级的日志。

如果你没有在你的应用程序中运行另一个Twisted reactor，这个的功能比 `CrawlerRunner` 更加合适。

CrawlerProcess对象必须通过 `Settings` 对象实例化。

**参数**：**install\_root_handler** - 是否安装root日志处理器(默认：True)

这个类不需要使用，除非编写脚本手动处理爬取进程。
</br></br>

**crawl(crawler\_or\_spidercls, \*args, \*\*kwargs)**


通过提供的参数运行crawler。

它将会调用给定的 Crawler类的 `crawl()` 方法，同时跟踪它，以便稍后停止。

如果 **crawler\_or_spidercls** 不是一个 `Crawler` 实例，这个方法将尝试使用这个参数创建一个spider类。

返回一个爬取结束时触发的deferred。

&nbsp;&nbsp;&nbsp;**参数**：

  - **crawler_or_spidercls**(`Crawler` 实例，`Spider` 子类，或者 字符串) - 已创建的crawler，或者spider的子类，或者项目中的spider的名字。
  - **args**(列表) - 实例化spider的参数
  - **kwargs**(字典) - 实例化spider的参数


**crawlers**

设置通过`crawl()` 方法开启，通过这个类管理的 `crawlers`。

**create_crawler(crawler\_or\_spidercls)**

返回一个`Crawler` 对象。

 - 如果 crawler\_or\_spidercls 是Crawler，按原样返回。
 - 如果 crawler\_or\_spidercls 是Spider子类，用这个子类构造新的Crawler。
 - 如果 crawler\_or\_spidercls 是字符串，这个函数在Scrapy项目中寻找名字为这个字符串的spider(使用spider加载器)，然后用这儿spider创建一个Crawler示例。

**join()**

返回当所有管理的 `crawler` 都完成了触发的deferred。

**start(stop\_after\_crawl=True)**

这个方法运行Twisted reactor，调整它的池尺寸到 ` REACTOR_THREADPOOL_MAXSIZE`，安装一个基于`DNSCACHE_ENABLED`和`DNSCACHE_SIZE`DNS缓存。

如果**stop\_after\_crawl 是 True，当所有crawler爬取完毕后，这个reactor使用 `join()` 停止。
</br>**参数**：**stop\_after\_crawl**(布尔值) - 当爬取完毕后是否停止reactor。

**stop()**

同时停止所有正在进行的爬行作业。

返回在它们全部结束时触发的deferred。

#Settings API#
 <table><tr><td>
scrapy.settings.SETTINGS_PRIORITIES
</td></tr></table>

设置Scrapy中使用的默认settings优先级的键名和优先级的字典。

每项定义了设置的入口点，为其提供了用作表示的代码名称和整数的优先级。当在`Settings`中设置和检索设置的值的时候，更高的优先级先于低优先级。

	SETTINGS_PRIORITIES = {
	    'default': 0,
	    'command': 10,
	    'project': 20,
	    'spider': 30,
	    'cmdline': 40,
	}

 <table><tr><td>
scrapy.settings.get_settings_priority(priority)
</td></tr></table>
一个小的辅助函数，在`SETTINGS_PRIORITIES`字典中查阅给定字符串的优先级，返回优先级的数字，或者直接返回一个给定的数字值。

<table><tr><td>
<font color=green>class</font>  scrapy.settings.Settings(values=None, priority='project')
</td></tr></table>
基类：`scrapy.settings.BaseSettings`

这个对象存储了用于配置内部组件的Scrapy设置，可以用作任何进一步的自定义。

它是`BaseSettings` 的直接子类，支持 `BaseSettings` 的所有方法。此外，这个类实例化后，新的对象将拥有已经产生的全局默认设置(内置的设置)。

 <table><tr><td>
<font color=green>class</font>    scrapy.settings.BaseSettings(values=None, priority='project')
</td></tr></table>

这个类的实例就像一个字典，但是是吧优先级连同<font color=red>`(key, value)`</font>一起存储，并且可以被冻结(即标记为不可变)。

键值条目可以在初始化的时候传递给 <font color=red>`values`</font> 参数，而且需要带上他们的 <font color=red>`priority` </font>优先级(除非 `values` 是一个 `BaseSettings` 的实例，这种情况已有的优先级将会被保留)。如果  <font color=red>`priority` </font>优先级参数是一个字符串，这个优先级名字将会在 `SETTINGS_PRIORITIES` 中寻找。否则，需要提供一个特定的整数。

一旦对象创建了，新的设置可以使用 `set()` 方法加载与更新，可以使用字典的方括号表示法，或者实例的 `get()` 方法对值进行访问。当给定一个键时，最高优先级的值将会被取出。

**copy()**

对当前设置深拷贝(deep copy)。

这个方法返回一个新的 `Settings` 类实例，具有与当前设置相同的值和优先级。

改变新的对象不会映射到原setting里面。

**copy\_to\_dict()**

拷贝当前的设置并转换为一个字典。

这个方法返回一个新的字典，具有与当前设置相同的值和优先级。

改变字典不会映射到原始的设置上。

这个方法可以在例如在Scrapy shell中打印设置使用到。

**freeze()**

禁用对当前设置的进一步更改。

在调用这个方法后，当前设置的状态将变成不可变的。尝试通过 `set()` 方法改变值报错。

**frozencopy()**

返回一个不可变的当前设置的拷贝。

`copy()` 的返回值调用 `freeze()` 的别名。

**get(name, default=None)**

获取某项配置的值，且不修改其原有的值。


&nbsp;&nbsp;&nbsp;&nbsp;**参数**：
	
  - **name** (字符串) – 配置名
  - **default** (任何) – 如果没有该项配置时返回的缺省值


**getbool(name, default=False)**

return False 将某项配置的值以布尔值形式返回。比如，<font color=red>`1`</font> 和 <font color=red>`'1'`</font>，<font color=red>`True`</font> 都返回<font color=red>`True`</font>， 而 <font color=red>`0`</font>，<font color=red>`'0'`</font>，<font color=red>`False`</font> 和 <font color=red>`None`</font> 返回 <font color=red>`False`</font>。


比如，通过环境变量计算将某项配置设置为<font color=red>`'0'`</font>，通过该方法获取得到 <font color=red>`False`</font>。

&nbsp;&nbsp;&nbsp;&nbsp;**参数**：
	
  - **name** (字符串) – 配置名
  - **default** (任何) – 如果没有该项配置时返回的缺省值


**getdict(name, default=None)**

获取一个设置的值，并且以字典的形式。如果你设置项的原始类型是一个字典，将会拷贝字典返回。如果是字符串(JSON格式的字符串)，它将会被转化为JSON字典。当设置项是一个`BaseSettings`实例，它会被这个实例转化为字典，包含所有它当前的设置的值，它们也可以通过`get()` 返回，丢失所有的优先级和可变性的信息(不可变的得到的也为可变)。


&nbsp;&nbsp;&nbsp;&nbsp;**参数**：
	
  - **name** (字符串) – 配置名
  - **default** (任何) – 如果没有该项配置时返回的缺省值


**getfloat(name, default=0.0)**

将某项配置的值以浮点数形式返回

&nbsp;&nbsp;&nbsp;&nbsp;**参数**：
	
  - **name** (字符串) – 配置名
  - **default** (任何) – 如果没有该项配置时返回的缺省值


**getint(name, default=0)**

将某项配置的值以整数形式返回

&nbsp;&nbsp;&nbsp;&nbsp;**参数**：
	
  - **name** (字符串) – 配置名
  - **default** (任何) – 如果没有该项配置时返回的缺省值


**getlist(name, default=None)**

将某项配置的值以列表形式返回。如果配置值本来就是list则原样返回。 如果是字符串，则返回被 ”,” 分割后的列表。

比如，某项值通过环境变量的计算被设置为 <font color=red>`'one,two'`</font> ，该方法返回<font color=red>`['one', 'two']`</font>。

&nbsp;&nbsp;&nbsp;&nbsp;**参数**：
	
  - **name** (字符串) – 配置名
  - **default** (任何) – 如果没有该项配置时返回的缺省值

**getpriority(name)**

返回当前设置的优先级的数值，给定的<font color=red>`name`</font> 不村子的时候返回 <font color=red>`None`</font>。

&nbsp;&nbsp;&nbsp;&nbsp;**参数**：
	
  - **name** (字符串) – 配置名

**maxpriority()**

返回所有设置中优先级最高的优先级数值，当没有存储任何设置时，返回 `SETTINGS_PRIORITIES` 的 <font color=red>`default`</font> 的优先级值(默认为 0 )

**getwithbase(name)**

获取类字典设置选项机器_BASE对应项的组合。

&nbsp;&nbsp;&nbsp;&nbsp;**参数**：
	
  - **name** (字符串) – 类字典配置的名称。

**set(name, value, priority='project')**

存储一个与给定的优先级的key/value键值对。

配置应该在配置Crawler对象(通过 `configure()` 方法)前产生，否则它们不会有任何效果。

&nbsp;&nbsp;&nbsp;&nbsp;**参数**：
	
  - **name** (字符串) – 配置名
  - **value**(any) - 与设置相关的值。
  - **priority**(string or int) - 设置的优先级。应该是 `SETTINGS_PRIORITIES` 的键或者一个整数。


**setmodule(module, priority='project')**

存储从一个给定优先级的模块中的设置项。(例如settings.py文件的设置项)

&nbsp;&nbsp;&nbsp;&nbsp;**参数**：

  - **module**(模块对象或者字符串) - 模块护着模块的路径
  - **priority**(字符串或者整数) - 设置的优先级。应该是 `SETTINGS_PRIORITIES` 的键或者一个整数。


**update(values, prority='project')**

存储给定优先级的键值对。

这个函数是用提供的 <font color=red>`priority`</font> 调用每个item(即`SETTINGS_PRIORITIES`的键)的 `set()` 方法。

如果<font color=red>`values`</font> 是字符串，它会被假定为一个JSON-decoded字符串，然后通过<font color=red>`json.loads()`</font> 方法被解析为字典。如果是一个 `BaseSettings` 实例，每个键的优先级将会被使用，<font color=red>`priority`</font> 参数将会被忽略。这个允许在一个命令下插入 / 更新设置和优先级。

&nbsp;&nbsp;&nbsp;&nbsp;**参数**：

  - **value**(字典或者字符串或者`BaseSettings`实例) - 设置的名和值
  - **priiority**(字符串或者整数) - 设置的优先级。应该是 `SETTINGS_PRIORITIES` 的键或者一个整数。


##SpiderLoader API##

 <table><tr><td>
<font color=green>class</font>    scrapy.loader.SpiderLoader
</td></tr></table>

该类负责检索和处理项目中定义的spider类。

可以在 `SPIDER_LOADER_CLASS` 的项目设置中指定自定义的spider loaders的路径，然后可以使用它。它们必须完全实现 `scrapy.interface.ISpiderLoader` 的接口来保证运行时不出错。

**from_settings(settings)**

这个类方法被Scrapy用来创建这个类的实例。它通过当前项目设置作位参数调用，递归加载 `SPIDER_MODULES` 中发现的spider。

&nbsp;&nbsp;&nbsp;&nbsp;**参数**：

  - **settings**(`Settings`实例) – 项目设置。

**load(spider_name)**

获得给定名字的Spider类。它会在先前加载的spider中寻找这个名字的spider，没有找到触发 *KeyError* 错误。

&nbsp;&nbsp;&nbsp;&nbsp;**参数**：

  - **spider_name**(str) – spider 类名。


**list()**

获得项目中可用spider的名字的列表。

**find\_by\_request(request)**

列出可以处理给定请求的Spider的名字。将会尝试将request的url与spider的allowed_domains匹配。


&nbsp;&nbsp;&nbsp;&nbsp;**参数**：

  - **request**(`Request` 实例) - 需要的请求。


##Signals API##

 <table><tr><td>
<font color=green>class</font>    scrapy.signalmanager.SignalManager(sender=_Anonymous)
</td></tr></table>

**connect(receiver, signal, \*\*kwargs)**

链接一个接收器函数(receiver function) 到一个信号(signal)。

signal可以是任何对象，虽然Scrapy提供了一些预先定义好的信号， 参考文档 信号(Signals)。

&nbsp;&nbsp;&nbsp;&nbsp;**参数**：

  - **receiver**(可调用对象) – 被链接到的函数
  - **signal**(对象) – 链接的信号

**disconnect(receiver, signal, \*\*kwargs)**

解除一个接收器函数和一个信号的关联。这跟方法 `connect()` 有相反的作用， 参数也相同。

**disconnect_all(signal, \*\*kwargs)**

取消给定信号绑定的所有接收器。

&nbsp;&nbsp;&nbsp;&nbsp;**参数**：

  - **signal**(对象) – 解除链接的信号


**send\_catch_log(signal, \*\*kwargs)**

发送一个信号，捕获异常并记录日志。

关键字参数会传递给信号处理者(signal handlers)(通过方法 `connect()` 关联)。

**send\_catch_log\_deferred(signal, \*\*kwargs)**

跟 `send_catch_log()` 相似但支持返回 deferreds 形式的信号处理器。

返回一个 deferred ，当所有的信号处理器的延迟被触发时调用。 发送一个信号，处理异常并记录日志。

关键字参数会传递给信号处理者(signal handlers)(通过方法 `connect()` 关联)。

##Stats Collector API##

模块 `scrapy.statscollectors` 下有好几种状态收集器， 它们都实现了状态收集器API对应的类 `StatsCollector` (即它们都继承至该类)。

 <table><tr><td>
<font color=green>class</font>    scrapy.statscollectors.StatsCollector
</td></tr></table>

**get_value(key, default=None)**

返回指定key的统计值，如果key不存在则返回缺省值。

**get_stats()**

以dict形式返回当前spider的所有统计值。

**set_value(key, value)**

设置key所指定的统计值为value。

**set_stats(stats)**

使用dict形式的 <font color=red>`stats`</font> 参数覆盖当前的统计值。

**inc_value(key, count=1, start=0)**

增加key所对应的统计值，增长值由count指定。 如果key未设置，则使用start的值设置为初始值。

**max_value(key, value)**

如果key所对应的当前value小于参数所指定的value，则设置value。 如果没有key所对应的value，设置value。

**min_value(key, value)**

如果key所对应的当前value大于参数所指定的value，则设置value。 如果没有key所对应的value，设置value。

**clear_stats()**

清除所有统计信息。

###以下方法不是统计收集api的一部分，但实现自定义的统计收集器时会使用到：###

**open_spider(spider)**

打开指定spider进行统计信息收集。

**close_spider(spider)**

关闭指定spider。调用后，不能访问和收集统计信息。
