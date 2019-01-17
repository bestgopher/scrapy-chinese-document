#调试内存溢出(Debugging memory leaks)#

在Scrapy中，类似Requests, Response及Items的对象具有有限的生命周期: 他们被创建，使用，最后被销毁。

这些对象中，Request的生命周期应该是最长的，其会在调度队列(Scheduler queue)中一直等待，直到被处理。 更多内容请参考 架构概览 。

由于这些Scrapy对象拥有很长的生命，因此将这些对象存储在内存而没有正确释放的危险总是存在。 而这导致了所谓的”内存泄露”。

为了帮助调试内存泄露，Scrapy提供了跟踪对象引用的机制，叫做 trackref ， 或者您也可以使用第三方提供的更先进内存调试库 Guppy (更多内容请查看下面)。而这都必须在 Telnet终端 中使用。

##(Common causes of memory leaks)##

内存泄露经常是由于Scrapy开发者在Requests中(有意或无意)传递对象的引用(例如，使用 meta 属性或request回调函数)，使得该对象的生命周期与 Request的生命周期所绑定。这是目前为止最常见的内存泄露的原因， 同时对新手来说也是一个比较难调试的问题。

在大项目中，spider是由不同的人所编写的。而这其中有的spider可能是有”泄露的”， 当所有的爬虫同时运行时，这些影响了其他(写好)的爬虫，最终，影响了整个爬取进程。

内存泄露可能存在与一个您编写的中间件，管道(pipeline) 或扩展，在代码中您没有正确释放 (之前分配的)资源。例如，在`spider_opened`分配资源，但是没有在`spider_closed`释放资源，在你在单一进程运行多个spider时可能会造成问题。

##请求太多了?(Too Many Requests?)##

默认情况下，Scrapy把请求队列放在内存中；它包含了`Request`对象和Request对象属性(例如`meta`中的对象)关联的对象。虽然不一定会溢出，但是可能会占用很多内存。启用 persistent job queue 可以帮助控制内存的使用。

##使用 <font color=red>`trackref`</font> 调试内存泄露(Debugging memory leaks with <font color=red>`trackref`</font>)##

`trackref` 是Scrapy提供用于调试大部分内存泄露情况的模块。 简单来说，其追踪了所有活动(live)的Request, Request, Item及Selector对象的引用。

您可以进入telnet终端并通过 <font color=red>`prefs()`</font> 功能来检查多少(上面所提到的)活跃(alive)对象。 pref() 是 `print_live_refs()` 功能的引用:

	telnet localhost 6023
	
	>>> prefs()
	Live References
	
	ExampleSpider                       1   oldest: 15s ago
	HtmlResponse                       10   oldest: 1s ago
	Selector                            2   oldest: 0s ago
	FormRequest                       878   oldest: 7s ago

正如所见，报告也展现了每个类中最老的对象的时间(age)。
如果您有内存泄露，那您能找到哪个spider正在泄露的机会是查看最老的request或response。 您可以使用 `get_oldest()` 方法来获取每个类中最老的对象， 正如此所示(在终端中)(原文档没有样例)。

##哪些对象被追踪了?(Which objects are tracked?)##

<font color=red>`trackref`</font> 追踪的对象包括以下类(及其子类)的对象:

  - `scrapy.http.Request`
  - `scrapy.http.Response`
  - `scrapy.item.Item`
  - `scrapy.selector.Selector`
  - `scrapy.spiders.Spider`

##真实例子(A real example)##

让我们来看一个假设的具有内存泄露的准确例子。
假如我们有些spider的代码中有一行类似于这样的代码:

	return Request("http://www.somenastyspider.com/product.php?pid=%d" % product_id,
	    callback=self.parse, meta={referer: response})

代码中在request中传递了一个response的引用，使得reponse的生命周期与request所绑定， 进而造成了内存泄露。

让我们来看看如何使用 <font color=red>`trackref`</font> 工具来发现原因(当然是在不知道任何的前提的情况下)。

当crawler运行了一小阵子后，我们发现内存占用增长了很多。 这时候我们进入telnet终端，查看活跃(live)的引用:

	>>> prefs()
	Live References
	
	SomenastySpider                     1   oldest: 15s ago
	HtmlResponse                     3890   oldest: 265s ago
	Selector                            2   oldest: 0s ago
	Request                          3878   oldest: 250s ago

上面具有非常多的活跃(且运行时间很长)的response，而其比Request的时间还要长的现象肯定是有问题的。响应的数量和请求的数量差不多，因此看起来它们在某种程度上是相互关联的。现在我们可以查看该spider的代码并发现导致泄露的那行代码(在request中传递response的引用)。

有时候关于活跃对象的额外信息是非常有帮助的。我们检查一下最老的响应：

	>>> from scrapy.utils.trackref import get_oldest
	>>> r = get_oldest('HtmlResponse')
	>>> r.url
	'http://www.somenastyspider.com/product.php?pid=123'

如果您想要遍历所有而不是最老的对象，您可以使用 `scrapy.utils.trackref.iter_all()` 方法:

	>>> from scrapy.utils.trackref import iter_all
	>>> [r.url for r in iter_all('HtmlResponse')]
	['http://www.somenastyspider.com/product.php?pid=123',
	 'http://www.somenastyspider.com/product.php?pid=584',
	...

##很多spider?(Too many spiders?)##

如果您的项目有很多的spider在并行执行，`prefs()` 的输出会变得很难阅读。针对于此， 该方法具有 ignore 参数，用于忽略特定的类(及其子类)。例如，将不会展现任何spider类的活跃引用。
	
	>>> from scrapy.spiders import Spider
	>>> prefs(ignore=Spider)

###scrapy.utils.trackref module###
以下是 trackref 模块中可用的方法。

<table><tr><td>
<font color=green>class</font> scrapy.utils.trackref.object_ref
</td></tr></table>

如果您想通过 <font color=red>`trackref`</font> 模块追踪活跃的实例，继承该类(而不是对象)。

<table><tr><td>
scrapy.utils.trackref.print_live_refs(class_name, ignore=NoneType)
</td></tr></table>

打印活跃引用的报告，以类名分类。</br>
**参数**：</br>
  
  - ignore (类或者类的元组) – 如果给定，所有指定类(或者类的元组)的对象将会被忽略。

<table><tr><td>
scrapy.utils.trackref.get_oldest(class_name)
</td></tr></table>

返回给定类名的最老活跃(alive)对象，如果没有则返回 <font color=red>`None`</font> 。首先使用 `print_live_refs()` 来获取每个类所跟踪的所有活跃(live)对象的列表。

##使用Guppy调试内存泄露(Debugging memory leaks with Guppy)##

<font color=red>`trackref`</font> 提供了追踪内存泄露非常方便的机制，其仅仅追踪了比较可能导致内存泄露的对象 (Requests, Response, Items及Selectors)。然而，内存泄露也有可能来自其他(更为隐蔽的)对象。 如果是因为这个原因，通过 <font color=red>`trackref`</font> 则无法找到泄露点，您仍然有其他工具: Guppy library 。如果你使用的是python3，请看 Debugging memory leaks with muppy。

如果你使用的<font color=red>`pip`</font>，你可以通过以下的命令安装Guppy。

	pip install guppy

telnet终端也提供了快捷方式(<font color=red>`hpy`</font>)来访问Guppy堆对象(heap objects)。 下面给出了通过使用Guppy查看堆中所有可用的Python对象的例子:

	>>> x = hpy.heap()
	>>> x.bytype
	Partition of a set of 297033 objects. Total size = 52587824 bytes.
	 Index  Count   %     Size   % Cumulative  % Type
	     0  22307   8 16423880  31  16423880  31 dict
	     1 122285  41 12441544  24  28865424  55 str
	     2  68346  23  5966696  11  34832120  66 tuple
	     3    227   0  5836528  11  40668648  77 unicode
	     4   2461   1  2222272   4  42890920  82 type
	     5  16870   6  2024400   4  44915320  85 function
	     6  13949   5  1673880   3  46589200  89 types.CodeType
	     7  13422   5  1653104   3  48242304  92 list
	     8   3735   1  1173680   2  49415984  94 _sre.SRE_Pattern
	     9   1209   0   456936   1  49872920  95 scrapy.http.headers.Headers
	<1676 more rows. Type e.g. '_.more' to view.>

您可以看到大部分的空间被字典所使用。接着，如果您想要查看哪些属性引用了这些字典， 您可以：

	>>> x.bytype[0].byvia
	Partition of a set of 22307 objects. Total size = 16423880 bytes.
	 Index  Count   %     Size   % Cumulative  % Referred Via:
	     0  10982  49  9416336  57   9416336  57 '.__dict__'
	     1   1820   8  2681504  16  12097840  74 '.__dict__', '.func_globals'
	     2   3097  14  1122904   7  13220744  80
	     3    990   4   277200   2  13497944  82 "['cookies']"
	     4    987   4   276360   2  13774304  84 "['cache']"
	     5    985   4   275800   2  14050104  86 "['meta']"
	     6    897   4   251160   2  14301264  87 '[2]'
	     7      1   0   196888   1  14498152  88 "['moduleDict']", "['modules']"
	     8    672   3   188160   1  14686312  89 "['cb_kwargs']"
	     9     27   0   155016   1  14841328  90 '[1]'
	<333 more rows. Type e.g. '_.more' to view.>

如上所示，Guppy模块十分强大，不过也需要一些关于Python内部的知识。关于Guppy的更多内容请参考 [Guppy documentation](http://guppy-pe.sourceforge.net/).

###使用muppy调试内存泄露(Debugging memory leaks with muppy)###

如果你使用的是Python3，你可以从 [Pympler](https://pypi.org/project/Pympler/) 中使用muppy。

如果你使用的是 <font color=red>`pip`</font>，你可以使用以下命令安装muppy。

	pip install Pympler

下面给出了通过使用muppy查看堆中所有可用的Python对象的例子:

	>>> from pympler import muppy
	>>> all_objects = muppy.get_objects()
	>>> len(all_objects)
	28667
	>>> from pympler import summary
	>>> suml = summary.summarize(all_objects)
	>>> summary.print_(suml)
	                               types |   # objects |   total size
	==================================== | =========== | ============
	                         <class 'str |        9822 |      1.10 MB
	                        <class 'dict |        1658 |    856.62 KB
	                        <class 'type |         436 |    443.60 KB
	                        <class 'code |        2974 |    419.56 KB
	          <class '_io.BufferedWriter |           2 |    256.34 KB
	                         <class 'set |         420 |    159.88 KB
	          <class '_io.BufferedReader |           1 |    128.17 KB
	          <class 'wrapper_descriptor |        1130 |     88.28 KB
	                       <class 'tuple |        1304 |     86.57 KB
	                     <class 'weakref |        1013 |     79.14 KB
	  <class 'builtin_function_or_method |         958 |     67.36 KB
	           <class 'method_descriptor |         865 |     60.82 KB
	                 <class 'abc.ABCMeta |          62 |     59.96 KB
	                        <class 'list |         446 |     58.52 KB
	                         <class 'int |        1425 |     43.20 KB
	
关于muppy的更多信息，参考[muppy文档](https://pythonhosted.org/Pympler/muppy.html)。

##Leaks without leaks##

有时候，您可能会注意到Scrapy进程的内存占用只在增长，从不下降。不幸的是， 有时候这并不是Scrapy或者您的项目在泄露内存。这是由于一个已知(但不有名)的Python问题。 Python在某些情况下可能不会返回已经释放的内存到操作系统。关于这个问题的更多内容请看:

  - [Python Memory Management](http://www.evanjones.ca/python-memory.html)
  - [Python Memory Management Part 2](http://www.evanjones.ca/python-memory-part2.html)
  - [Python Memory Management Part 3](http://www.evanjones.ca/python-memory-part3.html)

改进方案由Evan Jones提出，在 这篇文章 中详细介绍，在Python 2.5中合并。 不过这仅仅减小了这个问题，并没有完全修复。引用这片文章:

*不幸的是，这个patch仅仅会释放没有在其内部分配对象的区域(arena)。这意味着 碎片化是一个大问题。某个应用可以拥有很多空闲内存，分布在所有的区域(arena)中， 但是没法释放任何一个。这个问题存在于所有内存分配器中。解决这个问题的唯一办法是 转化到一个更为紧凑(compact)的垃圾回收器，其能在内存中移动对象。 这需要对Python解析器做一个显著的修改。*

为了使内存消耗合理，您可以将作业拆分为几个较小的作业，或者 不时启用持久作业队列(persistent job queue )和停止/启动蜘蛛。