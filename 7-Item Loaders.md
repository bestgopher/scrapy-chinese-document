#Item Loaders#

Item Loaders 提供了一个便捷的机制装载抓取的 Items。 虽然Items可以从它自己的类似字典（dictionary-like）的API得到所需信息 ,不过 Item Loaders提供了许多更加方便的API，这些API通过自动完成那些具有共通性的任务，可从抓取进程中得到这些信息, 比如预先解析提取到的原生数据。

换句话来解释, Items 提供了盛装抓取到的数据的*容器* , 而Item Loaders提供了构件 *装载* 该容器。

Item Loaders 被设计用来提供一个既弹性又高效简便的构件， 以扩展或重写爬虫或源格式(HTML, XML之类的)等区域的解析规则， 这将不再是后期维护的噩梦。

#用Item Loaders装载Items(Using Item Loaders to populate items)#

要使用Item Loader, 你必须先将它实例化. 你可以使用类似字典的对象(例如: Item or dict)来进行实例化, 或者不使用对象也可以, 当不用对象进行实例化的时候,Item会自动使用 `ItemLoader.default_item_class` 属性中指定的Item 类在Item Loader constructor中实例化。

然后,你开始收集数值到Item Loader时,通常使用 Selectors。 你可以在同一个item field 里面添加多个数值；Item Loader将知道如何用合适的处理函数来“添加”这些数值。

下面是在 Spider 中典型的Item Loader的用法, 使用 Items chapter 中声明的 Product item:

	from scrapy.loader import ItemLoader
	from myproject.items import Product
	
	def parse(self, response):
	    l = ItemLoader(item=Product(), response=response)
	    l.add_xpath('name', '//div[@class="product_name"]')
	    l.add_xpath('name', '//div[@class="product_title"]')
	    l.add_xpath('price', '//p[@id="price"]')
	    l.add_css('stock', 'p#stock]')
	    l.add_value('last_updated', 'today') # you can also use literal values
	    return l.load_item()

快速查看这些代码之后,我们可以看到发现 name 字段被从页面中两个不同的XPath位置提取:

  1. <font color=red>`//div[@class="product_name"]`</font>
  2. <font color=red>`//div[@class="product_title"]`</font>


换言之,数据通过用 `add_xpath()` 的方法,把从两个不同的XPath位置提取的数据收集起来. 这是将在以后分配给 <font color=red>`name`</font> 字段中的数据｡

之后,类似的请求被用于 <font color=red>`price`</font> 和 <font color=red>`stock`</font> 字段 (后者使用 CSS selector 和 `add_css()` 方法), 最后使用不同的方法 add_value() 对 <font color=red>`last_update`</font> 填充文本值( <font color=red>`today`</font> ).

最终, 当所有数据被收集起来之后, 调用 `ItemLoader.load_item()` 方法, 实际上填充并且返回了之前通过调用 `add_xpath()`, `add_css()`, `and add_value()` 所提取和收集到的数据的 Item。

#Input and Output processors#

Item Loader在每个(Item)字段中都包含了一个输入处理器和一个输出处理器｡ 输入处理器收到数据时立刻提取数据 (通过 `add_xpath()`, `add_css()` 或者 `add_value()` 方法) 之后输入处理器的结果被收集起来并且保存在ItemLoader内. 收集到所有的数据后, 调用 `ItemLoader.load_item()` 方法来填充,并得到填充后的 Item 对象. 这时当输出处理器被和之前收集到的数据(和用输入处理器处理的)被调。输出处理器的结果是被分配到Item的最终值｡

让我们看一个例子来说明如何输入和输出处理器被一个特定的字段调用(同样适用于其他field):

	l = ItemLoader(Product(), some_selector)
	l.add_xpath('name', xpath1) # (1)
	l.add_xpath('name', xpath2) # (2)
	l.add_css('name', css) # (3)
	l.add_value('name', 'test') # (4)
	return l.load_item() # (5)

发生了这些事情:

 1. 从 <font color=red>`xpath1`</font> 提取出的数据,传递给 输入处理器 的 <font color=red>`name`</font> 字段.输入处理器的结果被收集和保存在Item Loader中(但尚未分配给该Item)｡
 2. 从 <font color=red>`xpath2`</font> 提取出来的数据,传递给(1)中使用的相同的 输入处理器 .输入处理器的结果被附加到在(1)中收集的数据(如果有的话)。
 3. 这种情况与前面的一个例子类似，除了数据是通过 <font color=red>`css`</font> CSS选择器提取之外，(1)和(2)使用相同的输入处理器(即 <font color=red>`xpath`</font>)。得到的结果追加到(1)和(2)的结果的集合中。
 4. 这情况也与前面的一个例子类似，除了值被直接声明而不是通过XPath表达式或者CSS选择器提取。然而，这个值仍然会通过了输入处理器。这种情况下，因为值是不可迭代的，在通过输入处理器之前它会被转化为一个单元素的可迭代对象，因为输入处理器总是接受可迭代对象。
 5. 数据通过(1)，(2)，(3)，(4)步收集完毕后通过输出处理器给 <font color=red>`name`</font> 字段。输出处理器的结果就是给 item 的 <font color=red>`name`</font> 字段的声明的值。

值得注意的是处理器仅仅是可执行的对象，调用它以解析数据，返回一个解析后的值。因此你可以使用任何函数作为输入或者输出处理器。唯一的要求就是它们必须接受一个(且仅仅只能一个)位置参数，这个参数是一个可迭代对象。

<font color=#0099ff>
NOTE：

输入与输出处理器必须接受一个迭代器作为它们的第一个参数。输出处理器可以为任何函数。输入处理器的结果将追加到一个内部的列表中(在Loader中)，这个列表包含收集到的值(为此字段)。输出处理器的结果是最终声明给字段的值。
</font>

如果你想要使用一个普通函数作为处理器，确定它接受的第一个参数为 <font color=red>`self`</font>。

	def lowercase_processor(self, values):
	    for v in values:
	        yield v.lower()
	
	class MyItemLoader(ItemLoader):
	    name_in = lowercase_processor

这是因为无论何时一个函数作为类变量被声明，它就变成一个方法，在调用的时候接受实例作为第一个参数。

你需要记住另一件事，就是输入处理器返回的值在内部被收集(列表中)，然后传递给输出处理器，构建成字段。

最后，但不是最重要的，为了方便起见，Scrapy内奸了一些常用的处理器。

##声明Item Loaders(Declaring Item Loaders)##

声明 Item Loaders 就像声明 Item 一样，使用类语法。例如：

from scrapy.loader import ItemLoader
from scrapy.loader.processors import TakeFirst, MapCompose, Join

	class ProductLoader(ItemLoader):
	
	    default_output_processor = TakeFirst()
	
	    name_in = MapCompose(unicode.title)
	    name_out = Join()
	
	    price_in = MapCompose(unicode.strip)
	
	    # ... 

正如你看见的一样，输入处理器使用 <font color=red>`_in`</font> 后缀声明，输出处理器使用 <font color=red>`_out`</font> 后缀声明。你也可以声明默认的输入/输出处理器，通过使用`ItemLoader.default_input_processor` 和 `ItemLoader.default_output_processor` 属性。

##声明输入和输出处理器(Declaring Input and Output Processors)##

如上一节所示，输入和输出处理器可以在Item Loader的定义域中声明，这是一种非常常见声明处理器的方式。然而，还有一个地方你可以指定输入和输入处理器：在Item Field元数据(metadata)中。例如：

	import scrapy
	from scrapy.loader.processors import Join, MapCompose, TakeFirst
	from w3lib.html import remove_tags
	
	def filter_price(value):
	    if value.isdigit():
	        return value
	
	class Product(scrapy.Item):
	    name = scrapy.Field(
	        input_processor=MapCompose(remove_tags),
	        output_processor=Join(),
	    )
	    price = scrapy.Field(
	        input_processor=MapCompose(remove_tags, filter_price),
	        output_processor=TakeFirst(),
	    )

>

	>>> from scrapy.loader import ItemLoader
	>>> il = ItemLoader(item=Product())
	>>> il.add_value('name', [u'Welcome to my', u'<strong>website</strong>'])
	>>> il.add_value('price', [u'&euro;', u'<span>1000</span>'])
	>>> il.load_item()
	{'name': u'Welcome to my website', 'price': u'1000'}

输入和输出处理器的优先顺序如下：

  1. Item Loader特别指定的字段属性：<font color=red>`field_in`</font> 和 <font color=red>`field_out`</font>(最高优先级)。
  2. 字段元数据(<font color=red>`input_processor`</font> 和 <font color=red>`output_processor`</font> 键)。
  3. Item Loader默认的处理器：`ItemLoader.default_input_processor`()和`ItemLoader.default_output_processor()`(最低优先级)。

##Item Loader上下文##

Item Loader的上下文是一个任意的键值对的字典，在Item Loader的任意输入和输出处理器中共享。它在Item Loader的声明，实例化和使用的时候传递。他们被用来改变输入和输出处理器的行为。

例如，假设你有一个 <font color=red>`parse_length`</font> 函数，它接受一个文本值，从这个文本值中提取它的长度：

	def parse_length(text, loader_context):
	    unit = loader_context.get('unit', 'm')
	    # ... length parsing code goes here ...
	    return parsed_length 

这个函数通过接受 <font color=red>`loader_context`</font> 参数显式地告诉Item Loader它可以接受一个 Item Loader 上下文，所以当Item Loader被调用时传递给这个函数当前激活的上下文，这个处理器函数(本例中的<font color=red>`parse_length`</font>)可以使用这些上下文。

这里有一些方式改变Item Loader上下文的值：

 1.通过改变当前激活的Item Loader上下文(`context` 属性)：

	loader = ItemLoader(product)
	loader.context['unit'] = 'cm'

 2.Item Loader实例化的时候(Item Loader的构造函数中的关键字参数存储在Item Loader上下文中)

	loader = ItemLoader(product, unit='cm')
 
 3.声明Item Loader的时候，支持通过Item Loader上下文实例化的输入/输出处理器。`MapCompose` 就是其中一个：

	class ProductLoader(ItemLoader):
    	length_out = MapCompose(parse_length, unit='cm')

##ItemLoader对象(ItemLoader objects)##

<table>
<tr>
<td>
<font color=green>class</font> scrapy.loader.ItemLoader([item, seletor, response,]**kwargs)
</td>
</tr>
</table>

返回一个新的Item Loader来填充给定的item。如果没有指定item，自动实例化类属性 `default_item_class`关联的类。

当通过 *selector* 或者 *response* 参数实例化`ItemLoader` 类的时候，提供了通过 **selector** 从网页中提取数据的便利机制。

**Parameters**：

 - **item**(`Item` 对象) - 接受Item的实例，后续调用 `add_xpath()`，`add_css()`，`add_value()`来填充 item 实例。
 - **selector**(`Selector`对象) - 从这个选择器实例中提取数据，使用 `add_xpath()/add_css()`或者`replace_xpath()/replace_css()`方法。
 - **response**(`Response` 对象) - response对象通过`default_selector_class`属性指定的Selector类构造selector对象，如果selector参数给定了，这个参数会被忽略。

item，selector，response和其余的关键字参数分配到Item Loader上下文中(使用`context`属性可以访问。)

`ItemLoader`实例有以下方法：

<font color=green>`get_value(value, *processors, **kwargs)`</font>：

&nbsp;&nbsp;&nbsp;通过给定的  <font color=red>`processors`</font> 和关键字参数处理给定的 <font color=red>`value`</font> 值。
  
&nbsp;&nbsp;&nbsp;可用的关键字参数：

&nbsp;&nbsp;&nbsp;re=字符串或者编译后的正则表达式-使用`extract_regex()`方法通过给定的正则表达式从给定的value值中提取数据，在处理器使用之前应用。

&nbsp;&nbsp;&nbsp;例如:

	>>> from scrapy.contrib.loader.processor import TakeFirst
	>>> loader.get_value(u'name: foo', TakeFirst(), unicode.upper, re='name: (.+)')
	'FOO`

<font color=green>`add_value(field_name, value, *processors, **kwargs)`</font>：

&nbsp;&nbsp;&nbsp;处理<font color=red>`value`</font>然后给指定的字段赋值。</br>&nbsp;&nbsp;&nbsp;这个值首先传递给 `get_value()`方法，<font color=red>`processors`</font>和<font color=red>`kwargs`</font>传递给`get_value()`处理，然后传递给字段的输入处理器，最后将收集到的结果赋值给字段。如果这个字段已经包含了收集到的数据，新数据将会追加。</br>&nbsp;&nbsp;&nbsp;给定的<font color=red>`field_name`</font>可以为<font color=red>`None`</font>，这种情况下可以添加多个字段的值。
值需要为一个字段名和值相映射的字典。

	loader.add_value('name', u'Color TV')
	loader.add_value('colours', [u'white', u'blue'])
	loader.add_value('length', u'100')
	loader.add_value('name', u'name: foo', TakeFirst(), re='name: (.+)')
	loader.add_value(None, {'name': u'foo', 'sex': u'male'})

<font color=green>`replace_value(field_name, value, *processors, **kwargs)`</font>：

&nbsp;&nbsp;&nbsp;与`add_value()`类似，但是此方法是替换已经收集到的数据，而不是追加。

<font color=green>`get_xpath(xpath, *processors, **kwargs)`</font>：

&nbsp;&nbsp;&nbsp;类似于`get_value()`，但是接受XPath而不是值，XPath被用来从selector中提取数据，返回unicode字符串的列。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Parameters**：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**xpath**(str)： 提取数据的XPath字符串

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**re**(str或者编译好的正则表达式)：通过正则表达式从XPath提取出来的数据中再次匹配。

举例:

	# HTML snippet: <p class="product-name">Color TV</p>
	loader.get_xpath('//p[@class="product-name"]')
	# HTML snippet: <p id="price">the price is $1200</p>
	loader.get_xpath('//p[@id="price"]', TakeFirst(), re='the price is (.*)')



<font color=green>`add_xpath(field_name, xpath, *processors, **kwargs)`</font>：

&nbsp;&nbsp;&nbsp;类似于`add_value()`，但是接受XPath而不是值，XPath被用来从selector中提取数据，返回unicode字符串的列表。</br>&nbsp;&nbsp;&nbsp;<font color=red>`kwargs`</font>如`get_xpath()`一样。</br></br>举例:

	# HTML snippet: <p class="product-name">Color TV</p>
	loader.add_xpath('name', '//p[@class="product-name"]')
	# HTML snippet: <p id="price">the price is $1200</p>
	loader.add_xpath('price', '//p[@id="price"]', re='the price is (.*)')


<font color=green>`replace_xpath(field_name, xpath, *processors, **kwargs)`</font>：

&nbsp;&nbsp;&nbsp;与`add_xpath()`类似，但是此方法是替换已经收集到的数据，而不是追加。

<font color=green>`get_css(css, *processors, **kwargs)`</font>：

&nbsp;&nbsp;&nbsp;类似于`get_value()`，但是接受CSS选择器而不是值，CSS选择器被用来从selector中提取数据，返回unicode字符串的列表。。</br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Parameters**：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**css**(str)： 提取数据的css选择器

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**re**(str或者编译好的正则表达式)：通过正则表达式从XPath提取出来的数据中再次匹配。

举例:

	# HTML snippet: <p class="product-name">Color TV</p>
	loader.get_css('p.product-name')
	# HTML snippet: <p id="price">the price is $1200</p>
	loader.get_css('p#price', TakeFirst(), re='the price is (.*)')

<font color=green>`add_css(field_name, css, *processors, **kwargs)`</font>：

&nbsp;&nbsp;&nbsp;类似于`add_value()`，但是CSS选择器而不是值，CSS选择器被用来从selector中提取数据，返回unicode字符串的列表。</br>&nbsp;&nbsp;&nbsp;<font color=red>`kwargs`</font>如`get_css()`一样。</br></br>

举例:

	# HTML snippet: <p class="product-name">Color TV</p>
	loader.add_css('name', 'p.product-name')
	# HTML snippet: <p id="price">the price is $1200</p>
	loader.add_css('price', 'p#price', re='the price is (.*)')

<font color=green>`replace_css(field_name, css, *processors, **kwargs)`</font>：

&nbsp;&nbsp;&nbsp;与`add_css()`类似，但是此方法是替换已经收集到的数据，而不是追加。

<font color=green>`load_item()`</font>：

&nbsp;&nbsp;&nbsp;使用目前为止收集到的数据填充item，然后返回item。收集到的数据首先通过输出处理器，得到最终的值，分配给每个item字段。

<font color=green>`nested_xpath(xpath)`</font>：

&nbsp;&nbsp;&nbsp;通过XPath selector创建一个嵌套的Loader。提供的selector相对于关联的ItemLoader的selector。内嵌的Loader共享父`ItemLoader`的`Item`，因此调用`add_xpath()`，`add_value()`，`replace_value()`会像预期的那样。

<font color=green>`nested_css(css)`</font>：

&nbsp;&nbsp;&nbsp;通过CSS selector创建一个嵌套的Loader。提供的selector相对于关联的ItemLoader的selector。内嵌的Loader共享父`ItemLoader`的`Item`，因此调用`add_xpath()`，`add_value()`，`replace_value()`会像预期的那样。


<font color=green>`get_collected_values(filed_name)`</font>：

&nbsp;&nbsp;&nbsp;返回指定字段收集的值。

<font color=green>`get_output_value(filed_name)`</font>：

&nbsp;&nbsp;&nbsp;返回指定字段通过输出处理器处理后的值。这个方法不会填充或者修改item。

<font color=green>`get_input_processor(filed_name)`</font>：

&nbsp;&nbsp;&nbsp;返回指定字段的输入处理器。

<font color=green>`get_output_processor(filed_name)`</font>：

&nbsp;&nbsp;&nbsp;返回指定字段的输出处理器。

<font color=green>`item`</font>：

&nbsp;&nbsp;&nbsp;通过Item Loader解析后的item。


<font color=green>`context`</font>：

&nbsp;&nbsp;&nbsp;Item Loader当前激活的上下文。


<font color=green>`default_item_class`</font>：

&nbsp;&nbsp;&nbsp;一个`Item`类，Item Loader构造的时候没有指定item，使用这个类实例化得到item。

<font color=green>`default_input_processor`</font>：

&nbsp;&nbsp;&nbsp;字段没有指定输入处理器时使用这个输入处理器。

<font color=green>`default_out_processor`</font>：

&nbsp;&nbsp;&nbsp;字段没有指定输出处理器时使用这个输出处理器。

<font color=green>`default_seletor_class`</font>：

&nbsp;&nbsp;&nbsp;如果在构造Item Loader只提供了reponse，
没有给selector参数，就使用这个参数指定的`Selector`类为`ItemLoader`构造selector实例。如果给了selector实例，这个参数会被忽略。这个属性有时候会在子类中被重写。

<font color=green>`selector`</font>：

&nbsp;&nbsp;&nbsp;提取数据的`Selector`对象。它是构造Item Loader给定的selector参数，如果只给定了response参数的话，它是`default_selector_class`通过response得到的selector。
这个属性应该是只读的(别修改)。

##内嵌Loader(Nested Loaders)##

当从一批文档中提取相关值的时候，可以创建一个内嵌Loader。想象以下你从网页的footer中提取数据的时候，就像下面的情况一样:

	<footer>
	    <a class="social" href="https://facebook.com/whatever">Like Us</a>
	    <a class="social" href="https://twitter.com/whatever">Follow Us</a>
	    <a class="email" href="mailto:whatever@example.com">Email Us</a>
	</footer>

没有内嵌Loaders，你需要为每个你想要提取的值指定完全的xpath(或者css)。

例如:

	loader = ItemLoader(item=Item())
	# load stuff not in the footer
	loader.add_xpath('social', '//footer/a[@class = "social"]/@href')
	loader.add_xpath('email', '//footer/a[@class = "email"]/@href')
	loader.load_item()

除此之外，你可以在footer选择器的基础上创建一个内嵌的Loader，相对于footer取值。这个功能和前面的一样，但是避免了重复footer选择器。

举例：

	loader = ItemLoader(item=Item())
	# load stuff not in the footer
	footer_loader = loader.nested_xpath('//footer')
	footer_loader.add_xpath('social', 'a[@class = "social"]/@href')
	footer_loader.add_xpath('email', 'a[@class = "email"]/@href')
	# no need to call footer_loader.load_item()
	loader.load_item()

你可以随意创建内嵌Loader，通过它使用XPath或者CSS选择器工作。作为一般的准则，使用内嵌的Loader是为了让你的代码更加简单，但是不能再这个上面走极端，或者让你的解析语句变得很难去阅读。

#重用和扩展Item Loaders(Reusing and extending Item Loader)#

当你的项目变得越来越大，爬虫也越来越多，维护变成一个基本的问题，特别是你不得不为每个爬虫处理许多不同的解析规则，有许多异常，但是也想要重用常用的处理器。

Item Loader被设计减轻解析规则的维护负担，同时不失去灵活性，同时，提供了扩展和覆盖它们的便捷机制。因为这个原因，Item Loader 支持传统的Python类继承，以解决特定爬虫(一组爬虫)的区别。

例如，假设一些特别的网站的产品名在三个破折号中(例如---Plasma TV---)，但是你不希望在最后爬取的产品名中有这些破折号。

这里演示了怎么通过扩展和重用默认的Product Item Loader(ProductLoader)移除这些破折号。

	from scrapy.loader.processors import MapCompose
	from myproject.ItemLoaders import ProductLoader
	
	def strip_dashes(x):
	    return x.strip('-')
	
	class SiteSpecificLoader(ProductLoader):
	    name_in = MapCompose(strip_dashes, ProductLoader.name_in)

扩展Item Loader在另一种情况下非常有用就是，当你有许多源格式，例如XML和HTML。在XML版本中你想移除<font color=red>`CDATA`</font>。这里演示了怎么做：

	from scrapy.loader.processors import MapCompose
	from myproject.ItemLoaders import ProductLoader
	from myproject.utils.xml import remove_cdata
	
	class XmlProductLoader(ProductLoader):
	    name_in = MapCompose(remove_cdata, ProductLoader.name_in)

这就是通常你扩展处理器的方式。

作为输出处理器，常见的声明方式是在字段的元数据中，因为它们通常依赖于特定的字段，而不是特定的网站解析规则(如输入处理器那样)。

还有一些其他的方式扩展、继承和重写你的Item Loaders,不同种类的Item Loader适合不同的项目。Scrapy只提供了这样的机制；它不是必须强加的Loader的特定部分-它视你和你的项目而定。

##可用的内建处理器(Available built-in processors)##

即便你可以使用任何可调用的函数最为输入和输出处理器，Scrapy还是提供了一些常用的处理器，如下描述所示。他们中，像`MapCompose`(通常作为输入处理器)把几个函数组合起来，按顺序执行这些函数，产生最终解析的值。

这里列出了所有内奸的处理器:

<table>
<tr>
<td>
<font color=green>class</font> scrapy.loader.processors.Identity
</td>
</tr>
</table>

最简单的处理器，不会做任何事。它返回没有改变过的原始值。它不用接受任何构造参数，也不接受Loader上下文。

例如：

	>>> from scrapy.loader.processors import Identity
	>>> proc = Identity()
	>>> proc(['one', 'two', 'three'])
	['one', 'two', 'three']


<table>
<tr>
<td>
<font color=green>class</font> scrapy.loader.processors.TakeFirst
</td>
</tr>
</table>

从接收到的值中返回第一个非空的值，通常用作单值字段的输出处理器。不接受构造参数，也不接受Loader上下文。

例如：

	>>> from scrapy.loader.processors import TakeFirst
	>>> proc = TakeFirst()
	>>> proc(['', 'one', 'two', 'three'])
	'one'

<table>
<tr>
<td>
<font color=green>class</font> scrapy.loader.processors.Join(separator=u'')
</td>
</tr>
</table>

返回通过构造函数给定的连接符连接而成的值，连接符默认为<font color=red>`u''`</font>。不接受Loader上下文。当使用默认连接符时，这个处理器相当于函数：<font color=red>`u''.join`</font>。

举例：

	>>> from scrapy.loader.processors import Join
	>>> proc = Join()
	>>> proc(['one', 'two', 'three'])
	u'one two three'
	>>> proc = Join('<br>')
	>>> proc(['one', 'two', 'three'])
	u'one<br>two<br>three'


<table>
<tr>
<td>
<font color=green>class</font> scrapy.loader.processors.Compose(*function, **default_loader_context)
</td>
</tr>
</table>

通过给定的函数构造而成的处理器。这就意味着每个输入的值传递个第一个函数，第一个函数的返回值传递给第二个函数，...，直到最后一个函数返回值的最为这个处理器的输出值。

默认情况下，<font color=red>`None`</font>值时处理器停止。这个行为可以通过传递位置参数 <font color=red>`stop_on_none=False`</font> 改变。

举例：

	>>> from scrapy.loader.processors import Compose
	>>> proc = Compose(lambda v: v[0], str.upper)
	>>> proc(['hello', 'world'])
	'HELLO'

每个函数都可以随意接受<font color=red>`loader_context`</font>参数。如果这样做，这个处理器将通过这个参数传递当前激活的Loader上下文。

这个关键字参数在构造函数中传递，被用作默认的Loader上下文传递给没个调用的函数。然而，最终传递个函数的Loader上下文的值将会被当前激活的Loader上下文覆盖，这个当前激活的上下文就是`ItemLoader.context`属性。


<table>
<tr>
<td>
<font color=green>class</font> scrapy.loader.processors.MapCompose(*function, **default_loader_context)
</td>
</tr>
</table>

通过给定函数构造而成的处理器，与`Compose`处理器类似。不同点是内部函数之间所得结果的传递方式，描述如下：

处理器输入的值被遍历，第一个函数处理遍历出的每一个元素。这个函数调用的结果(每个元素都调用一次)合在一起构建成一个新的可迭代对象，然后把结果传递给第二个函数处理，以此类推，直到最后一个函数对收集来的值处理了为止。最后一个函数的输出值串在一起生成处理器的输出。

每个特定的函数能够返回一个值或者一列值，相同的函数返回的一列值将被展开，这适用于其他输入值。函数也可以返回 <font color=red>`None`</font>，这种情况下函数的None将会被忽略或者丢弃。

这个处理器提供了一个便捷的途径去组合函数来处理单个值，而不是整个可迭代对象。因为这个原因 `MapCompose` 通常被用来作为输入处理器，因为数据通常使用 selector 的 `extract()` 方法提取，这个方法返回就是unicode字符串的列表。

下面例子阐明它如何工作：
	
	>>> def filter_world(x):
	...     return None if x == 'world' else x
	...
	>>> from scrapy.loader.processors import MapCompose
	>>> proc = MapCompose(filter_world, unicode.upper)
	>>> proc([u'hello', u'world', u'this', u'is', u'scrapy'])
	[u'HELLO, u'THIS', u'IS', u'SCRAPY']

和 Compose 处理器一样，函数能够接受Loader上下文，构造函数的关键字参数被用作默认的上下文值。

<table>
<tr>
<td>
<font color=green>class</font> scrapy.loader.processors.SelectJmes(json_path)
</td>
</tr>
</table>

通过提供的构造器，使用json路径查询数据，并返回输出。需要 `jmespath` (https://github.com/jmespath/jmespath.py)模块运行。这个处理器一次只接受一个输入。

举例：

	>>> from scrapy.loader.processors import SelectJmes, Compose, MapCompose
	>>> proc = SelectJmes("foo") #for direct use on lists and dictionaries
	>>> proc({'foo': 'bar'})
	'bar'
	>>> proc({'foo': {'bar': 'baz'}})
	{'bar': 'baz'}

处理Json：

	>>> import json
	>>> proc_single_json_str = Compose(json.loads, SelectJmes("foo"))
	>>> proc_single_json_str('{"foo": "bar"}')
	u'bar'
	>>> proc_json_list = Compose(json.loads, MapCompose(SelectJmes('foo')))
	>>> proc_json_list('[{"foo":"bar"}, {"baz":"tar"}]')
	[u'bar']

