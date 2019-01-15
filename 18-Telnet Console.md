#Telnet Console#

Scrapy提供了内置的telnet终端，以供检查，控制Scrapy运行的进程。 telnet仅仅是一个运行在Scrapy进程中的普通python终端。因此您可以在其中做任何事。

telnet终端是一个 自带的Scrapy扩展 。 该扩展默认为启用，不过您也可以关闭。 关于扩展的更多内容请参考 Telnet console 扩展(Telnet console Extension) 。

##如何访问telnet终端(How to access the telnet console)##

telnet终端监听设置中定义的 `TELNETCONSOLE_PORT` ，默认为 <font color=red>`6023`</font> 。 访问telnet请输入:

	telnet localhost 6023
	>>>

Windows及大多数Linux发行版都自带了所需的telnet程序。

##telnet终端中可用的变量(Available variables in the telnet console)##

telnet仅仅是一个运行在Scrapy进程中的普通python终端。因此您可以做任何事情，甚至是导入新终端。

telnet为了方便提供了一些默认定义的变量：

  - **crawler**		- Scrapy Crawler (`scrapy.crawler.Crawler` 对象)
  - **engine**		- Crawler.engine属性
  - **spider**		- 当前激活的爬虫(spider)
  - **slot** - the engine slot
  - **extensions** - 扩展管理器(manager) (Crawler.extensions属性)
  - **stats** - 	状态收集器 (Crawler.stats属性)
  - **settings** - Scrapy设置(setting)对象 (Crawler.settings属性)
  - **est** - 打印引擎状态的报告
  - **prefs** - 针对内存调试 (参考 调试内存溢出)
  - **p** - pprint.pprint 函数的简写
  - **hpy** - 	针对内存调试 (参考 调试内存溢出)


#Telnet console 使用示例(Telnet console usage examples)#

下面是使用telnet终端的一些例子:

##查看引擎状态(View engine status)##

在终端中您可以使用Scrapy引擎的 <font color=red>`est()`</font> 方法来快速查看状态:

	telnet localhost 6023
	>>> est()
	Execution engine status
	
	time()-engine.start_time                        : 8.62972998619
	engine.has_capacity()                           : False
	len(engine.downloader.active)                   : 16
	engine.scraper.is_idle()                        : False
	engine.spider.name                              : followall
	engine.spider_is_idle(engine.spider)            : False
	engine.slot.closing                             : False
	len(engine.slot.inprogress)                     : 16
	len(engine.slot.scheduler.dqs or [])            : 0
	len(engine.slot.scheduler.mqs)                  : 92
	len(engine.scraper.slot.queue)                  : 0
	len(engine.scraper.slot.active)                 : 0
	engine.scraper.slot.active_size                 : 0
	engine.scraper.slot.itemproc_size               : 0
	engine.scraper.slot.needs_backout()             : False


##暂停，恢复和停止Scrapy引擎(Pause, resume and stop the Scrapy engine)##

暂停:

	telnet localhost 6023
	>>> engine.pause()
	>>>

恢复:

	telnet localhost 6023
	>>> engine.unpause()
	>>>

停止:

	telnet localhost 6023
	>>> engine.stop()
	Connection closed by foreign host.

#Telnet终端信号(Telnet Console signals)#

<table><tr><td>
scrapy.extensions.telnet.update_telnet_vars(telnet_vars)
</td></tr></table>

在telnet终端开启前发送该信号。您可以挂载(hook up)该信号来添加，移除或更新 telnet本地命名空间可用的变量。 您可以通过在您的处理函数(handler)中更新 <font color=red>`telnet_vars`</font> 字典来实现该修改。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**参数**：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**telnet_vars**(dict)： telnet变量的字典


##Telnet settings##

以下是终端的一些设定：

<font color=green>TELNETCONSOLE_PORT</font>

默认：<font color=red>`[6023, 6073]`</font>

telnet终端使用的端口范围。如果设为 <font color=red>`None`</font> 或 <font color=red>`0`</font> ， 则动态分配端口。

<font color=green>TELNETCONSOLE_HOST</font>

默认：<font color=red>`'127.0.0.1'`</font>

telnet终端监听的接口(interface)。