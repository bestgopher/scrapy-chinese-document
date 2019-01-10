#Item Pipeline#

当Item在Spider中被收集之后，它将会被传递到Item Pipeline，一些组件会按照一定的顺序执行对Item的处理。

每个item pipeline组件(有时称之为“Item Pipeline”)是实现了简单方法的Python类。他们接收到Item并通过它执行一些行为，同时也决定此Item是否继续通过pipeline，或是被丢弃而不再进行处理。

以下是item pipeline的一些典型应用：
 
  - 清理HTML数据
  - 验证爬取的数据(检查item包含某些字段)
  - 查重(并丢弃)
  - 将爬取结果保存到数据库中

#编写你自己的item pipeline(Writing your own item)#

每个item pipiline组件是一个独立的Python类，同时必须实现以下方法:

<table>
<tr>
<td>
<font color=green>process_item(self, item, spider)</font>
</td>
</tr>
</table>
每个item pipeline组件都需要调用该方法。`process_item()`必须返回以下两者的一个：数据组成的字典或者`Item`(或者子类)对象，Twisted Deferred 或者抛出 `DropItem` 异常。被丢弃的item将不会被之后的pipeline组件所处理。

参数:	</br>
&nbsp;&nbsp;**item** (Item 对象或者字典) – 被爬取的item</br>
&nbsp;&nbsp;**spider** (Spider 对象) – 爬取该item的spider

此外,他们也可以实现以下方法:

<table>
<tr>
<td>
<font color=green>open_spider(self, spider)</font>
</td>
</tr>
</table>

当spider被开启时，这个方法被调用。

参数:	</br>
&nbsp;&nbsp;**spider** (`Spider` 对象) – 被开启的spider

<table>
<tr>
<td>
<font color=green>close_spider(self, spider)</font>
</td>
</tr>
</table>
当spider被关闭时，这个方法被调用

参数:	</br>&nbsp;&nbsp;**spider** (Spider 对象) – 被关闭的spider

<table>
<tr>
<td>
<font color=green>from_crawler(cls, crawler)</font>
</td>
</tr>
</table>
如果存在，这个类方法将会从`Crawler`中被调用，用以创建pipeline实例。它必须返回一个新的pipeline实例。Crawler提供所有Scrapy核心组件，例如settings和signals；这为Pipeline提供了接受这个组件，连接这些功能的方式。

参数:</br>
&nbsp;&nbsp;**crawler**(`Crawler`对象) - 使用这个pipeline的crawler。

#Item pipeline样例(Item pipeline example)#

##验证价格，同时丢弃没有价格的item(Price validation and dropping items with no prices)##

让我们来看一下以下这个假设的pipeline，它为那些不含税(<font color=red>`price_excludes_vat`</font> 属性)的item调整了 <font color=red>`price`</font> 属性，同时丢弃了那些没有价格的item:

	from scrapy.exceptions import DropItem
	
	class PricePipeline(object):
	
	    vat_factor = 1.15
	
	    def process_item(self, item, spider):
	        if item['price']:
	            if item['price_excludes_vat']:
	                item['price'] = item['price'] * self.vat_factor
	            return item
	        else:
	            raise DropItem("Missing price in %s" % item)

##将item写入JSON文件(Write items to a JSON file)##

以下pipeline将所有(从所有spider中)爬取到的item，存储到一个独立地 items.jl 文件，每行包含一个序列化为JSON格式的item:

	import json
	
	class JsonWriterPipeline(object):
	
	    def open_spider(self, spider):
	        self.file = open('items.jl', 'w')
	
	    def close_spider(self, spider):
	        self.file.close()
	
	    def process_item(self, item, spider):
	        line = json.dumps(dict(item)) + "\n"
	        self.file.write(line)
	        return item

<font color=#0000ff>
NOTE：</br>
JsonWriterPipeline的目的只是为了介绍怎样编写item pipeline，如果你想要将所有爬取的item都保存到同一个JSON文件， 你需要使用 Feed exports 。
</font>

##截屏item(Take screenshot of item)##

这个例子演示了怎么从`process_item()`方法中返回Deferred。它使用的是Splash渲染item中url的截取画面。pipeline向本地运行的Splash实例发送请求。然后请求被下载，Deffered回调fires，它保存item到一个文件中，向item添加文件名。

	import scrapy
	import hashlib
	from urllib.parse import quote
	
	
	class ScreenshotPipeline(object):
	    """Pipeline that uses Splash to render screenshot of
	    every Scrapy item."""
	
	    SPLASH_URL = "http://localhost:8050/render.png?url={}"
	
	    def process_item(self, item, spider):
	        encoded_item_url = quote(item["url"])
	        screenshot_url = self.SPLASH_URL.format(encoded_item_url)
	        request = scrapy.Request(screenshot_url)
	        dfd = spider.crawler.engine.download(request, spider)
	        dfd.addBoth(self.return_item, item)
	        return dfd
	
	    def return_item(self, response, item):
	        if response.status != 200:
	            # Error happened, return item.
	            return item
	
	        # Save screenshot to file, filename will be hash of url.
	        url = item["url"]
	        url_hash = hashlib.md5(url.encode("utf8")).hexdigest()
	        filename = "{}.png".format(url_hash)
	        with open(filename, "wb") as f:
	            f.write(response.body)
	
	        # Store filename in item.
	        item["screenshot_filename"] = filename
	        return item

##去重(Duplicates filter)##

一个用于去重的过滤器，丢弃那些已经被处理过的item。让我们假设我们的item有一个唯一的id，但是我们spider返回的多个item中包含有相同的id：

	from scrapy.exceptions import DropItem
	
	class DuplicatesPipeline(object):
	
	    def __init__(self):
	        self.ids_seen = set()
	
	    def process_item(self, item, spider):
	        if item['id'] in self.ids_seen:
	            raise DropItem("Duplicate item found: %s" % item)
	        else:
	            self.ids_seen.add(item['id'])
            return item

##启用一个Item Pipeline组件(Activating an Item Pipeline component)##

为了启用一个Item Pipeline组件，你必须将它的类添加到 ITEM_PIPELINES 配置，就像下面这个例子:

	ITEM_PIPELINES = {
	    'myproject.pipelines.PricePipeline': 300,
	    'myproject.pipelines.JsonWriterPipeline': 800,
	}

设置中为pipeline类声明的整数值决定了它们运行的顺序：item从值低到值高的顺序在pipeline中传递。习惯定义这个数字在 `0-1000` 范围内。
