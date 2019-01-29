#Feed exports#

*0.10 新版功能*。

实现爬虫时最经常提到的需求就是能合适的保存爬取到的数据，或者说，生成一个带有爬取数据的”输出文件”(通常叫做”输出feed”)，来供其他系统使用。

Scrapy自带了Feed输出，并且支持多种序列化格式(serialization format)及存储方式(storage backends)。

#序列化方式(Serialization formats)#

feed输出使用到了 Item exporters 。其自带支持的类型有:
  
 - JSON
 - JSON lines
 - CSV
 - XML

您也可以通过 `FEED_EXPORTERS` 设置扩展支持的属性。

##JSON##

   - `FEED_FORMAT`： <font color=red>`json`</font>
   - 使用的exporter: `JsonItemExporter`
   - 大数据量情况下使用JSON请参见 这个警告：

<font color=orange>
警告：</br>
JSON 是一个简单而有弹性的格式, 但对大量数据的扩展性不是很好，因为这里会将整个对象放入内存. 如果你要JSON既强大又简单,可以考虑 JsonLinesItemExporter , 或把输出对象分为多个块.</font>

##JSON lines##

  - `FEED_FORMAT`：<font color=red>`jsonlines`</font>
  - 使用的exporter： `JsonLinesItemExporter`

##CSV##

  - `FEED_FORMAT`：<font color=red>`xml`</font>
  - 使用的exporter：`XmlItemExporter`
  - 为了指定导出的列和列的顺序，在setting.py中指定`FEED_EXPORT_FIELDS`。其他的feed exporters也会使用这个选项，但是这个对CSV非常重要，因为不像其他的exports那样，CSV格式有一个固定的头。

##XML##
  - `FEED_FORMAT`： <font color=red>`xml`</font>
  - 使用的exporter： <font color=red>`XmlItemExporter`</font>

##Pickle##
  - `FEED_FORMA`T：<font color=red>`pickle`</font>
  - 使用的exporter：`PickleItemExporter`

##Marshal##
  - `FEED_FORMAT`：<font color=red>marshal</font>
  - 使用的exporter:`MarshalItemExporter`

#存储(Storages)#

使用feed输出时您可以通过使用 URI (通过 `FEED_URI` 设置) 来定义存储端。 feed输出支持URI方式支持的多种存储后端类型。

自带支持的存储后端有:
  - 本地文件系统
  - FTP
  - S3(需要boto或者botocore)
  - 标准输出

有些存储后端会因所需的外部库未安装而不可用。例如，S3只有在 boto  或者 botocore库安装的情况下才可使用(只有pyhton2的scrapy才支持boto)。

#存储URI参数(Storage URI parameters)#

存储URI也包含参数。当feed被创建时这些参数可以被覆盖：

  - <font color=red>`%(time)s`</font> - 被feed创建时的时间戳替换
  - <font color=red>`%(name)s`</font> - 被spider的名字替换

其他命名的参数会被spider同名的属性所覆盖。例如， 当feed被创建时， <font color=red>`%(site_id)s`</font> 将会被 <font color=red>`spider.site_id`</font> 属性所覆盖。

下面用一些例子来说明:

  - 存储在FTP，每个spider一个目录:</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color=red>`ftp://user:password@ftp.example.com/scraping/feeds/%(name)s/%(time)s.json`</font>
  - 存储在S3，每一个spider一个目录：</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color=red>`s3://mybucket/scraping/feeds/%(name)s/%(time)s.json`</font>

#存储端(Storage backends)#

##本地文件系统(Local filesystem)##

将feed存储在本地系统。

  - URI scheme：<font color=red>`file`</font>
  - URI样例: <font color=red>`file:///tmp/export.csv`</font>
  - 需要的外部依赖库：none

注意: (只有)存储在本地文件系统时，您可以指定一个绝对路径 <font color=red>`/tmp/export.csv`</font>并忽略协议(scheme)。不过这仅仅只能在Unix系统中工作。

##FTP##

将feed存储在FTP服务器。

  - URI scheme：ftp
  - URI样例：<font color=red>`ftp://user:pass@ftp.example.com/path/to/export.csv`</font>
  - 需要的外部依赖库：none

##S3##

将feed存储在 Amazon S3 。

  - URI scheme：<font color=red>`s3`</font>
  - Example URIs：</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color=red>`s3://mybucket/path/to/export.csv`</font>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color=red>`s3://aws_key:aws_secret@mybucket/path/to/export.csv`</font>
  - 需要的外部依赖库: botocore(pyhton2和python3)，或者boto(pyhton3)

您可以通过在URI中传递user/pass来完成AWS认证，或者也可以通过下列的设置来完成:</br>

  - AWS\_ACCESS\_KEY_ID
  - AWS\_SECRET\_ACCESS_KEY

##标准输出(Standard output)##

feed输出到Scrapy进程的标准输出。

  - URI scheme: <font color=red>`stdout`</font>
  - URI样例: <font color=red>`stdout:`</font>
  - 需要的外部依赖库: none

##设定(Settings)##

这些是配置feed输出的设定:

  - **FEED_URI** (必须)
  - **FEED_FORMAT**
  - **FEED_STORAGES**
  - **FEED\_EXPORTERS**
  - **FEED\_STORE_EMPTY**
  - **FEED\_EXPORT_ENCODING**
  - **FEED\_EXPORT_FIELDS**
  - **FEED\_EXPORT_INDENT**

##FEED_URI##

Default: <font color=red>`None`</font>

输出feed的URI。支持的URI协议请参见 存储端(Storage backends) 。
**为了启用feed输出，该设定是必须的。**

##FEED_FORMAT##

输出feed的序列化格式。可用的值请参见 序列化方式(Serialization formats) 。

##FEED\_EXPORT_ENCODING##

Default: <font color=red>`None`</font></br>
用来设置输出feed的编码方式。

如果不设置或者设置为 <font color=red>`None`</font>(默认)，除了JSON之外其他格式都会使用UTF-8编码，JSON因为历史原因使用安全的数字编码(<font color=red>`\uxxxx`</font> 序列)。

如果你想JSON使用UTF-8编码，设置为<font color=red>`utf-8`</font>。

##FEED\_EXPORT_FIELDS##
Default: <font color=red>`None`</font></br>

要导出的字段的列表，可选。例如：<font color=red>`FEED_EXPORT_FIELDS = ["foo", "bar", "baz"]`</font>。</br>
使用 FEED\_EXPORT\_FIELDS 选项定义导出的字段和顺序。</br>
当 FEED\_EXPORT\_FIELDS 为空或者 None (默认)，Scrapy使用spider生成的定义在字典或者 `Item` 子类中的字段。

如果exporter需要一个固定的字段顺序(CSV格式这种情况)，但是 FEED\_EXPORT\_FIELDS 是空的或者为None，Scrapy会尽量从需要导出的数据中推断出 - 一般使用第一个item的字段名。

##FEED\_EXPORT\_INDENT##

Default: <font color=red>`0`</font>

用于在输出的每个层级的缩进的空格数目。如果<font color=red>`FEED_EXPORT_INDENT`</font>是一个非负整数，数组的元素和对象的成员将会使用这个缩进层次打印得非常美观。缩进为 <font color=red>`0`</font> 的话(默认)，或者为复数的话，每个item将会从新的一行开始打印。<font color=red>`None`</font>选择最紧凑简洁的表现形式。

目前只有在 `JsonItemExporter` 和 `XmlItemExporter` 即格式为 <font color=red>`.json`</font> 和 <font color=red>`.xml`</font>中使用。

##FEED\_STORE\_EMPTY##

Default: <font color=red>`False`</font>

是否输出空feed(没有item的feed)。

##FEED_STORAGES##

Default：<font color=red>`{}`</font>

包含项目支持的额外feed存储端的字典。 字典的键(key)是URI协议(scheme)，值是存储类(storage class)的路径。

##FEED\_STORAGES\_BASE##

Default:

	{
	    '': 'scrapy.extensions.feedexport.FileFeedStorage',
	    'file': 'scrapy.extensions.feedexport.FileFeedStorage',
	    'stdout': 'scrapy.extensions.feedexport.StdoutFeedStorage',
	    's3': 'scrapy.extensions.feedexport.S3FeedStorage',
	    'ftp': 'scrapy.extensions.feedexport.FTPFeedStorage',
	}

一个包含Scrapy内置存储端的字典。你可以在 `FEED_STORAGES` 中定义它们的URI协议为 <font color=red>`None`</font> 来禁用它们。例如，禁用内置的FTP存储端(无需替换)，在你的 <font color=red>`settings.py`</font> 设置：

	FEED_STORAGES = {
	    'ftp': None,
	}


##FEED_EXPORTERS##

Default：<font color=red>`{}`</font>

包含项目支持的额外输出器(exporter)的字典。 该字典的键(key)是URI协议(scheme)，值是 Item输出器(exporter) 类的路径。

##FEED\_EXPORTERS\_BASE##

Default:

	{
	    'json': 'scrapy.exporters.JsonItemExporter',
	    'jsonlines': 'scrapy.exporters.JsonLinesItemExporter',
	    'jl': 'scrapy.exporters.JsonLinesItemExporter',
	    'csv': 'scrapy.exporters.CsvItemExporter',
	    'xml': 'scrapy.exporters.XmlItemExporter',
	    'marshal': 'scrapy.exporters.MarshalItemExporter',
	    'pickle': 'scrapy.exporters.PickleItemExporter',
	}

一个包含Scrapy内置feed exporters的字典。你可以在 `FEED_EXPORTERS` 中定义它们的序列化格式为 <font color=red>`None`</font> 来禁用它们。例如，禁用内置的CSV格式(无需替换)，在你的 <font color=red>`settings.py`</font> 设置：

	FEED_EXPORTERS = {
	    'csv': None,
	}