# 命令行工具(Command line tool) #

版本 0.10里面的新功能

Scrapy通过 <font color=red>`scrapy`</font> 命令行工具控制，这里称为'Scrapy tool'以区分'commands'或者'Scrapy commands'等子命令。

Scrapy tool为多种用途提供了各自的命令，每个命令接受参数或者选项作为不同的设置。

(<font color=red>`scrapy deploy`</font> 命令在1.0版本被移除，支持独立的<font color=red>`scrapyd-deploy`</font>)

## 配置设置(Configuration settings) ##

Scrapy将会在特定位置中的ini样式文件 <font color=red>`scrapy.cfg`</font> 中寻找配置参数：
  
  1. <font color=red>`/etc/scrapy.cfg`</font> 或者 <font color=red>`c:\scrapy\scrapy.cfg`</font> （系统范围），
  2. <font color=red>`~/.config/scrapy.cfg`</font> (<font color=red>`$XDG_CONFIG_HOME`</font>) 和 <font color=red>`~/.scrapy.cfg`</font> (<font color=red>`$HOME`</font>) 用于全局设置(用户范围)
  3. <font color=red>`scrapy.cfg`</font> 项目根目录下。

列出的设置将会按照以下优先级进行合并：用户定义的优先级高于系统默认的，如果项目层面有定义设置的话，将会覆盖所有已存在的。

Scrapy能够识别以下环境变量来设置：
	

  - <font color=red>`SCRAPY_SETTINGS_MODULE`</font>
  - <font color=red>`SCRAPY_PROJECT`</font>
  - <font color=red>`SCRAPY_PYTHON_SHELL`</font>


## scrapy项目的默认结构(Default structure of Scrapy projects) ##

深入钻研命令行工具和它的子命令之前，我们先了解Scrapy项目的目录结构。

虽然可能被修改，但是所有的Scrapy项目默认有着相同的目录结构，类似于：

	scrapy.cfg
	myproject/
	    __init__.py
	    items.py
	    middlewares.py
	    pipelines.py
	    settings.py
	    spiders/
	        __init__.py
	        spider1.py
	        spider2.py
	        ...

<font color=red>`scrapy.cfg`</font> 文件所在的目录被称为项目的根目录。scrapy.cfg文件包含python模块的名字，定义了项目的设置。如下所示：

	[settings]
	default = myproject.settings

## 使用<font color=red>`scrapy`</font> 工具(Using scrapy tool) ##

你可以在没有参数的情况下运行Scrapy工具，这将会打印一些使用帮助和可用的命令：

	Scrapy X.Y - no active project
	
	Usage:
	  scrapy <command> [options] [args]
	
	Available commands:
	  crawl         Run a spider
	  fetch         Fetch a URL using the Scrapy downloader
	[...]

如果你现在在一个项目之中，第一行将会打印当前的项目。上面示例中，没有在一个项目中运行的。如果在一个项目中运行，打印会如下所示：
	
	Scrapy X.Y - project: myproject
	
	Usage:
	  scrapy <command> [options] [args]
	
	[...]

## 创建项目(Creating project) ##

通常你用 <font color=red>`scray`</font> 工具做的第一件事就是创建项目：

    scrapy startproject myproject [project_dir]

这将会创建一个项目在 <font color=red>`project_dir`</font> 目录下。如果 <font color=red> `project_dir`</font> 没有指定， <font color=red>`project_dir`</font> 将会和
 <font color=red>`myproject`</font> 一样。

接下来，你进入新项目目录中：

	cd project_dir

这里你准备好使用 <font color=red>`scray`</font> 命令去管理和控制你的项目了。

## 控制项目(Controlling projects) ##

你使用你项目内部的 <font color=red>`scray`</font> 工具来管理和控制它们。

例如，创建一个新的爬虫：

    scray genspider mydomain mydomain.com

有一些命令(如 `crawl`) 必须在项目内运行。详细信息后面介绍。

还要记住的是，有些命令在项目中运行可能会有细微差别的行为。比如，<font color=red>`fetch`</font> 命令提取的url与某些爬虫相关联的话，则会使用这个爬虫的动作(比如`UA`会被覆盖)。这是有意为之的，<font color=red>`fetch`</font> 命令就是用来测试检查爬虫如何下载页面的。

## 可用的命令工具(Available tool commands) ##

本段列出了可用的内置命令，包含描述和一些有用的示例。记住，你可以通过运行以下的命令获得每个命令更多的信息：

	scrapy <command> -h

你也可以通过以下命令查看所有的命令：

	scrapy -h

有两种命令，一种是只能在Scrapy项目中运行的命令(项目特定命令)，一种是不需要项目中运行的命令(全局命令)，全局命令在项目中运行可能会有些细微的差别(因为项目设置会覆盖全局设置)。

全局命令：
 
 - `startproject`
 - `genspider`
 - `settings`
 - `runspider`
 - `shell`
 - `fetch`
 - `view`
 - `version`

项目特定命令:
 
 - `crawl`
 - `check`
 - `list`
 - `edit`
 - `parse`
 - `bench`


##startproject ##
 
 - 语法：<font color=red>`scrapy startproject <project_name> [project_dir]`</font>
 - 是否需要项目：不需要

在 <font color=red>`project_dir`</font> 目录下创建名为 <font color=red>`project_name`</font> 的项目。如果 <font color=red>`project_dir`</font> 没有指定， <font color=red>`project_dir`</font> 将会和
 <font color=red>`project_name`</font> 一样。

举例：

	$ scrapy startproject myproject

##genspider##

 - 语法： <font color=red>`scrapy genspider [-t template] <name> <domain>`</font>
 - 是否需要项目：不需要

在当前目录中创建一个新的爬虫，如果它在项目内被调用则会在当前项目的 <font color=red>`spiders`</font> 目录下创建爬虫。<font color=red>`name`</font> 参数是设置爬虫的名字，<font color=red>`domain`</font> 参数被用于生成爬虫的 <font color=red>`allowed_domains`</font> 和 <font color=red>`start_urls`</font>属性。

举例：

	$ scrapy genspider -l
	Available templates:
	  basic
	  crawl
	  csvfeed
	  xmlfeed
	
	$ scrapy genspider example example.com
	Created spider 'example' using template 'basic'
	
	$ scrapy genspider -t crawl scrapyorg scrapy.org
	Created spider 'scrapyorg' using template 'crawl'

这仅仅是在预定义好的模板上创建爬虫的方便的快捷方式而已，但不是创建爬虫的唯一方式。你可以自己书写爬虫文件而不是使用命令生成它。

## crawl ##

 - 语法：<font color=red>scrapy crawl <spider\></font>
 - 是否需要项目：需要

使用爬虫开始爬取。

举例：

	$ scrapy crawl myspider
	[ ... myspider starts crawling ... ]

##check##

 - 语法：<font color=red>`scrapy check [-l] <spider>`</font>
 - 是否需要项目：需要

运行contract检查

举例：

	$ scrapy check -l
	first_spider
	  * parse
	  * parse_item
	second_spider
	  * parse
	  * parse_item
	
	$ scrapy check
	[FAILED] first_spider:parse_item
	>>> 'RetailPricex' field is missing
	
	[FAILED] first_spider:parse
	>>> Returned 92 requests, expected 0..4

##list##
 - 语法：<font color=red>`scrapy list`</font>
 - 是否需要项目：需要

列出本项目中可用的爬虫。每行输出一个爬虫。

举例：

	$ scrapy list
	spider1
	spider2

##edit##
 - 语法：<font color=red>`scrapy edit <spider>`</font>
 - 是否需要项目：需要

使用环境变量中默认的编辑器编辑爬虫文件，如果环境变量中没有设定，使用配置文件中的 <font color=red>`EDITOR`</font> 变量指定的编辑器。

这个命令只是提供快捷的编辑方式，开发者当然也可以自由地选择任何IDE工具编写和调试爬虫。

举例：

    $ scrapy edit spider1

##fetch##

 - 语法：<font color=red>`scrapy fetch <url>`</font>
 - 是否需要项目：不需要

使用Scrapy下载器下载给定url的网页，标准输出网页源代码。

有趣的是这个命令会按照定义的爬虫的方式下载页面。例如，我们定义的了爬虫的 <font color=red>`USER_AGENT`</font> 属性，它就会使用这个属性。

因此此页面可以查看你的爬虫爬取了一个啥样的页面。

如果在项目外使用这个命令，没有特别指定行为的话，它将会使用默认的Scrapy下载器设置。

支持的选项：

 - <font color=red>`--spider=SPIDER`</font>：指定使用哪个爬虫爬取。
 - <font color=red>`--header`</font>：打印响应的请求头而不是请求体。
 - <font color=red>`--no-redirect`</font>：不允许网页重定向(默认允许)
 
举例：

	$ scrapy fetch --nolog http://www.example.com/some/page.html
	[ ... html content here ... ]
	
	$ scrapy fetch --nolog --headers http://www.example.com/
	{'Accept-Ranges': ['bytes'],
	 'Age': ['1263   '],
	 'Connection': ['close     '],
	 'Content-Length': ['596'],
	 'Content-Type': ['text/html; charset=UTF-8'],
	 'Date': ['Wed, 18 Aug 2010 23:59:46 GMT'],
	 'Etag': ['"573c1-254-48c9c87349680"'],
	 'Last-Modified': ['Fri, 30 Jul 2010 15:30:18 GMT'],
	 'Server': ['Apache/2.2.3 (CentOS)']}


##view##

 - 语法：<font color=red>`scrapy view <url>`</font>
 - 是否需要项目：不需要

用Scrapy获得url响应的源代码，通过浏览器显示出来。有时候，爬虫得到的网页和人们常规看到的不同，因此这个用来检查爬虫得到的网页是什么样的，网页是否是你期望的那样。

支持的选项：

 - <font color=red>`--spider=SPIDER`</font>：指定使用哪个爬虫爬取。
 - <font color=red>`--no-redirect`</font>：不允许网页重定向。

举例：
	
	$ scrapy view http://www.example.com/some/page.html
	[ ... browser starts ... ]


##shell##

 - 语法：<font color=red>`scrapy shell [url]`</font>
 - 是否需要项目：不需要

以给定的url(如果给定)或没有url启动shell。支持UNIX风格的本地文件路径，<font color=red>**./**</font> 或者 <font color=red>**../**</font>表示的相对路径和绝对路径。

支持的选项：

 - <font color=red>`--spider=SPIDER`</font>：指定使用哪个爬虫爬取。
 - <font color=red>`-c code`</font>：执行代码，打印结果并退出。
 - <font color=red>`--no-redirect`</font>：不允许网页重定向，只对开始shell的url有用。当进入shell中使用 <font color=red>`fetch(url)`</font> 提取网页时，依然允许网页重定向。

举例：

	$ scrapy shell http://www.example.com/some/page.html
	[ ... scrapy shell starts ... ]
	
	$ scrapy shell --nolog http://www.example.com/ -c '(response.status, response.url)'
	(200, 'http://www.example.com/')
	
	# shell follows HTTP redirects by default
	$ scrapy shell --nolog http://httpbin.org/redirect-to?url=http%3A%2F%2Fexample.com%2F -c '(response.status, response.url)'
	(200, 'http://example.com/')
	
	# you can disable this with --no-redirect
	# (only for the URL passed as command line argument)
	$ scrapy shell --no-redirect --nolog http://httpbin.org/redirect-to?url=http%3A%2F%2Fexample.com%2F -c '(response.status, response.url)'
	(302, 'http://httpbin.org/redirect-to?url=http%3A%2F%2Fexample.com%2F')

##parse##

 - 语法：<font color=red>`scrapy parse <url> [option]`</font>
 - 是否需要项目：需要

提取给定的url并用处理它的爬虫解析它。使用 <font color=red>`--callback`</font>选项指定解析方法，没有指定这个选项默认使用 <font color=red>`parse`</font> 方法。

支持的选项:

 - <font color=red>`--spider=SPIDER`</font>：指定使用哪个爬虫爬取。
 - <font color=red>`--a NAME=VALUE`</font>：设置爬虫的参数(可以重复指定多个参数)
 - <font color=red>`--callback`</font> or <font color=red>`-c`</font>：爬虫解析响应的回调方法
 - <font color=red>`--meta`</font> or <font color=red>`-m`</font>：给请求添加meta参数。它必须是一个合法的json字符串，例如: --meta='{"foo": "bar"}'
 - <font color=red>`--pipeline`</font>：通过pipeline处理item
 - <font color=red>`--rule`</font> or <font color=red>`-r`</font>：使用 **CrawlSpider** 规则匹配解析网页的回调方法(即爬虫的方法)。
 - <font color=red>`--noitem`</font>：不显示爬取的item
 - <font color=red>`--nolinks`</font>：不显示提取的链接
 - <font color=red>`--nocolour`</font>:避免使用 **pygments** 为输出着色。
 - <font color=red>`--depth`</font> or <font color=red>`-d`</font>：递归请求的深度级别(默认为1)
 - <font color=red>`--verbose`</font> or <font color=red>`-v`</font>：显示每个深度级别的信息

举例：

	$ scrapy parse http://www.example.com/ -c parse_item
	[ ... scrapy log lines crawling example.com spider ... ]
	
	>>> STATUS DEPTH LEVEL 1 <<<
	# Scraped Items  ------------------------------------------------------------
	[{'name': u'Example item',
	 'category': u'Furniture',
	 'length': u'12 cm'}]
	
	# Requests  -----------------------------------------------------------------
	[]


##settings##

 - 语法：<font color=red>`scrapy settings [options]`</font>
 - 是否需要项目：不需要

获得Scrapy设置的值

如果在项目中，则显示项目中的设置值，如果不是在项目中，显示Scrapy默认的值。

举例：

	$ scrapy settings --get BOT_NAME
	scrapybot
	$ scrapy settings --get DOWNLOAD_DELAY
	0

##runspider##

 - 语法：<font color=red>`scrapy runspider <spider_file.py>`</font>
 - 是否需要项目：不需要

运行一个没有创建项目的情况下自定义的爬虫文件。

举例：
	
	$ scrapy runspider myspider.py
	[ ... spider starts crawling ... ]

##version##

 - 语法：<font color=red>`scrapt version [-v]`</font>
 - 是否需要项目：不需要

打印Scrapy的版本。使用 <font color=red>`-v`</font> 选项，也会打印Python、Twisted和Platform信息。在bug报告中非常有用。

##bench##

版本0.17中新加

 - 语法：<font color=red>`scrapy bench`</font>
 - 是否需要项目：不需要

运行快速基准测试。Benchmarking.

##自定义项目命令(Custom project commands)##

你也可以通过使用 <font color=red>`COMMANDS_MODULE`</font> 设置添加自定义项目命令。

##COMMANDS_MODULE##

默认：<font color=red>`''`</font> 空字符串

用于查找添加自定义的Scrapy命令。用于为你的Scrapy项目添加自定义的命令。

举例：
	
	COMMANDS_MODULE = 'mybot.commands'

##通过setup.py入口点注册命令(Register commands via setup.py entry points)##


<font color=#0099ff>
NOTE：

这是实验性的功能，请谨慎使用。
</font>

你可以通过添加外部库的<font color=red>`setup.py`</font>文件的 <font color=red>`entry_ponits`</font> 参数添加 <font color=red>`scrapy.commands`</font> 选项来添加Scrapy命令。

下面命令是添加 <font color=red>`my_command`</font> 命令。

	from setuptools import setup, find_packages
	
	setup(name='scrapy-mymodule',
	  entry_points={
	    'scrapy.commands': [
	      'my_command=my_scrapy_module.commands:MyCommand',
	    ],
	  },
	 )


