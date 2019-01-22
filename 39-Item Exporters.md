#Item Exporters#

当你抓取了你要的数据(Items)，你就会想要将他们持久化或导出它们，并应用在其他的程序。这是整个抓取过程的目的。

为此，Scrapy提供了Item Exporters 来创建不同的输出格式，如XML，CSV或JSON。

##使用 Item Exporter(Using Item Exporters)##

如果你很忙，只想使用 Item Exporter 输出数据，请查看 Feed exports. 相反，如果你想知道Item Exporter 是如何工作的，或需要更多的自定义功能（不包括默认的 exports），请继续阅读下文。

为了使用 Item Exporter，你必须对 Item Exporter 及其参数 (args) 实例化。每个 Item Exporter 需要不同的参数，详细请查看 Item Exporters 参考资料 。在实例化了 exporter 之后，你必须：

  1. 调用方法 `start_exporting()` 以标识 exporting 过程的开始。
  2. 对要导出的每个项目调用 `export_item()` 方法。
  3. 最后调用 `finish_exporting()` 表示 exporting 过程的结束

这里你能看到一个item pipeline 使用多个Item Exporters来根据字段的值导出item。

	from scrapy.exporters import XmlItemExporter
	
	class PerYearXmlExportPipeline(object):
	    """Distribute items across multiple XML files according to their 'year' field"""
	
	    def open_spider(self, spider):
	        self.year_to_exporter = {}
	
	    def close_spider(self, spider):
	        for exporter in self.year_to_exporter.values():
	            exporter.finish_exporting()
	            exporter.file.close()
	
	    def _exporter_for_item(self, item):
	        year = item['year']
	        if year not in self.year_to_exporter:
	            f = open('{}.xml'.format(year), 'wb')
	            exporter = XmlItemExporter(f)
	            exporter.start_exporting()
	            self.year_to_exporter[year] = exporter
	        return self.year_to_exporter[year]
	
	    def process_item(self, item, spider):
	        exporter = self._exporter_for_item(item)
	        exporter.export_item(item)
	        return item

##序列化 item fields(Serialization of item fields)##
默认情况下，该字段值将不变的传递到序列化库，如何对其进行序列化的决定被委托给每一个特定的序列化库。

但是，你可以自定义每个字段值如何序列化在它被传递到序列化库中之前。

有两种方法可以自定义一个字段如何被序列化，请看下文。

####1. 在 field 类中声明一个 serializer(1. Declaring a serializer in the field)####

如果你使用 `item` 您可以在 field metadata 声明一个 serializer。该 serializer 必须可调用，并返回它的序列化形式。

例如：

	import scrapy
	
	def serialize_price(value):
	    return '$ %s' % str(value)
	
	class Product(scrapy.Item):
	    name = scrapy.Field()
	    price = scrapy.Field(serializer=serialize_price)

##2. 覆盖(overriding) serialize_field() 方法(Overriding the serialize_field() method)##

你可以覆盖 `serialize_field()` 方法来自定义如何输出你的数据。

在你的自定义代码后确保你调用父类的 `serialize_field()` 方法。

实例:

	from scrapy.exporter import XmlItemExporter
	
	class ProductXmlExporter(XmlItemExporter):
	
	    def serialize_field(self, field, name, value):
	        if field == 'price':
	            return '$ %s' % str(value)
	        return super(Product, self).serialize_field(field, name, value)

##内置Item Exporters 参考资料(Built-in Item Exporters reference)##

下面是一些Scrapy内置的 Item Exporters类. 其中一些包括了实例, 假设你要输出以下2个Items:

	Item(name='Color TV', price='1200')
	Item(name='DVD player', price='200')

####BaseItemExporter####

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.contrib.exporter.BaseItemExporter(fields_to_export=None, export_empty_fields=False, encoding='utf-8')
</td></tr></table>

这是一个对所有 Item Exporters 的(抽象)父类。它对所有(具体) Item Exporters 提供基本属性，如定义export什么fields, 是否export空fields, 或是否进行编码。

你可以在构造器中设置它们不同的属性值: `fields_to_export `, `export_empty_fields`, `encoding`, `indent`

**export_item(item)**

输出给定item. 此方法必须在子类中实现.

**serialize_field(field, name, value)**

返回给定field的序列化值. 你可以覆盖此方法来控制序列化或输出指定的field.

默认情况下, 此方法寻找一个 serializer 在 item field 中声明 并返回它的值. 如果没有发现 serializer, 则值不会改变，除非你使用<font color=red>`unicode`</font> 值并编码到 <font color=red>`str`</font>， 编码可以在 `encoding` 属性中声明.

&nbsp;&nbsp;&nbsp;**参数**：

  - **field** (`Field`对象或者空字典) – 被序列化的字段。如果exports的是一个字典而不是`Item`z，这个值是一个空字典。
  - **name** (str) – 被序列化的字段的名字
  - **value** – 被序列化的值

**start_exporting()**

表示exporting过程的开始. 一些exporters用于产生需要的头元素(例如 `XmlItemExporter`). 在实现exporting item前必须调用此方法.

**finish_exporting()**

表示exporting过程的结束. 一些exporters用于产生需要的尾元素 (例如 `XmlItemExporter`). 在完成exporting item后必须调用此方法.

**fields_to_export**

列出export什么fields值, None表示export所有fields. 默认值为None.

一些 exporters (例如` CsvItemExporter`) 按照定义在属性中fields的次序依次输出.

有些exporters可能需要 `fields_to_exports` 列表，为了当spider返回的是字典(而不是 `Item` 实例)的时候正确的导出数据。


**export_empty_fields**

是否在输出数据中包含为空的item fields. 默认值是 <font color=red>`False`</font>. 一些 exporters (例如 `CsvItemExporter`) 会忽略此属性并输出所有fields.

这个选项对字典忽略。

**encoding**

Encoding 属性将用于编码 unicode 值. (仅用于序列化字符串).其他值类型将不变的传递到指定的序列化库.

**indent**

用于输出级别上的缩进空格数量。默认为 <font color=red>`0`</font>。

  -  <font color=red>`indent=None`</font> - 选择最紧凑的表示，同一行中的所有项目都没有缩进
  -  <font color=red>`indent<=0`</font> - 每个item都在自己的行上，没有缩进
  -  <font color=red>`indent>0`</font> -  每个item在其自己的行上，使用提供的数值缩进


###XmlItemExporter###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.exporters.XmlItemExporter(file, item_element='item', root_element='items', **kwargs)
</td></tr></table>

以XML格式 exports Items 到指定的文件类。

&nbsp;&nbsp;&nbsp;**参数**：

  - **file** – 用于export数据的类文件对象。它的<font color=red>`write`</font> 方法应该可以接受 <font color=red>`bytes`</font>(磁盘文件需要以二进制的模式打开，<font color=red>`io.BytesIO`</font> 对象，等等)。
  - **root_element**(str) – XML 根元素名.
  - **item_element**(str) – XML item 的元素名.

构造器额外的关键字参数将传给 `BaseItemExporter` 构造器.

一个典型的 exporter 实例:

	<?xml version="1.0" encoding="utf-8"?>
	<items>
	  <item>
	    <name>Color TV</name>
	    <price>1200</price>
	 </item>
	  <item>
	    <name>DVD player</name>
	    <price>200</price>
	 </item>
	</items>

除了覆盖 `serialize_field()` 方法, 多个值的 fields 会转化每个值到 <font color=red>`<value>`</font> 元素.

例如, item:

	Item(name=['John', 'Doe'], age='23')

将被序列化为:

	<?xml version="1.0" encoding="utf-8"?>
	<items>
	  <item>
	    <name>
	      <value>John</value>
	      <value>Doe</value>
	    </name>
	    <age>23</age>
	  </item>
	</items>

###CsvItemExporter###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.exporters.CsvItemExporter(file, include_headers_line=True, join_multivalued=', ', **kwargs)
</td></tr></table>

输出 csv 文件格式. 如果添加 `fields_to_export` 属性, 它会按顺序定义CSV的列名. `export_empty_fields` 属性在此没有作用.

&nbsp;&nbsp;&nbsp;**参数**：

  - **file** – 用于export数据的类文件对象。它的<font color=red>`write`</font> 方法应该可以接受 <font color=red>`bytes`</font>(磁盘文件需要以二进制的模式打开，<font color=red>`io.BytesIO`</font> 对象，等等)。
  - **include_headers_line** (str) – 启用后 exporter 会输出第一行为列名, 列名从 `BaseItemExporter.fields_to_export` 或第一个 item fields 获取.
  - **join_multivalued** – char 将用于连接多个值的fields.

此构造器额外的关键字参数将传给 BaseItemExporter 构造器 , 其余的将传给 csv.writer 构造器, 以此来定制 exporter.

一个典型的 exporter 实例:
	
	product,price
	Color TV,1200
	DVD player,200

###PickleItemExporter###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.exporters.PickleItemExporter(file, protocol=0, **kwargs)
</td></tr></table>

输出 pickle 文件格式.


&nbsp;&nbsp;&nbsp;**参数**：

  - **file** – 用于export数据的类文件对象。它的<font color=red>`write`</font> 方法应该可以接受 <font color=red>`bytes`</font>(磁盘文件需要以二进制的模式打开，<font color=red>`io.BytesIO`</font> 对象，等等)。
  - **protocol** (int) – pickle 协议.


更多信息请看 pickle module documentation.

此构造器额外的关键字参数将传给 BaseItemExporter 构造器.

Pickle 不是可读的格式，这里不提供实例.

###PprintItemExporter###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.contrib.exporter.PprintItemExporter(file, **kwargs)
</td></tr></table>

输出整齐打印的文件格式.

&nbsp;&nbsp;&nbsp;**参数**：

  - **file** – 用于export数据的类文件对象。它的<font color=red>`write`</font> 方法应该可以接受 <font color=red>`bytes`</font>(磁盘文件需要以二进制的模式打开，<font color=red>`io.BytesIO`</font> 对象，等等)。

一个典型的 exporter 实例:

	{'name': 'Color TV', 'price': '1200'}
	{'name': 'DVD player', 'price': '200'}

此格式会根据行的长短进行调整.

###JsonItemExporter###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.exporters.JsonItemExporter(file, **kwargs)
</td></tr></table>

输出 JSON 文件格式, 所有对象将写进一个对象的列表. 此构造器额外的关键字参数将传给 `BaseItemExporter` 构造器, 其余的将传给 JSONEncoder 构造器, 以此来定制 exporter.

&nbsp;&nbsp;&nbsp;**参数**：

  - **file** – 用于export数据的类文件对象。它的<font color=red>`write`</font> 方法应该可以接受 <font color=red>`bytes`</font>(磁盘文件需要以二进制的模式打开，<font color=red>`io.BytesIO`</font> 对象，等等)。

一个典型的 exporter 实例:

	[{"name": "Color TV", "price": "1200"},
	{"name": "DVD player", "price": "200"}]

<font color=orange>
WARNING:</br>

JSON 是一个简单而有弹性的格式, 但对大量数据的扩展性不是很好，因为这里会将整个对象放入内存. 如果你要JSON既强大又简单,可以考虑 `JsonLinesItemExporter` , 或把输出对象分为多个块.
</font>

###JsonLinesItemExporter###

<table><tr><td>
<font color=green>class</font>   &nbsp;scrapy.exporters.JsonLinesItemExporter(file, **kwargs)
</td></tr></table>
输出 JSON 文件格式, 每行写一个 JSON-encoded 项. 此构造器额外的关键字参数将传给 `BaseItemExporter `构造器, 其余的将传给 JSONEncoder 构造器, 以此来定制 exporter.

&nbsp;&nbsp;&nbsp;**参数**：

  - **file** – 用于export数据的类文件对象。它的<font color=red>`write`</font> 方法应该可以接受 <font color=red>`bytes`</font>(磁盘文件需要以二进制的模式打开，<font color=red>`io.BytesIO`</font> 对象，等等)。

一个典型的 exporter 实例:

{"name": "Color TV", "price": "1200"}
{"name": "DVD player", "price": "200"}

这个类能很好的处理大量数据.