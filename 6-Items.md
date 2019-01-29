#Items#

爬取的目的是从非结构化的资源中(网页)提取出结构化的数据。Scrapy能像Python字典一样返回提取的数据。虽然字典对我来说熟悉和方便，但是它缺少结构：很容易在字段名字上出错，或者返回不一致的数据，尤其是在有许多spider的大型项目中。

为了定义通用的输出数据格式，Scrapy提供了 `Item` 类。`Item` 对象是用来收集爬取的数据的简单容器。它们提供给了类似于字典的API，通过非常方便的语法声明可用的字段。

Scrapy的各个组件使用Items提供的额外信息：exporters通过声明的字段以确定要导出的列，可以自定义用来序列化字段元数据的方法，`trackref` 跟踪Item实例以帮助寻找内存泄漏。

##声明Items##

Item通过简单的类定义语法和 `Field` 类声明。例如:

	import scrapy
	
	class Product(scrapy.Item):
	    name = scrapy.Field()
	    price = scrapy.Field()
	    stock = scrapy.Field()
	    last_updated = scrapy.Field(serializer=str)

<font color=green>
`注意：熟悉Django的人会注意到Scrapy Item的声明类似于Django Models的声明，除此之外，Scrapy Items更加简单，因为没有各种不同字段类型的概念。
`</font>

##Item 字段(Item Fields)##

**`Field`** 对象指明了每个字段的元数据(metadata)。例如下面例子中 <font color=red>`last_updated`</font> 中指明了该字段的序列化函数。

你可以为每个字段指定各种类型的元数据。**`Field`** 对象对接受的值没有任何限制。也正是因为这个原因，文档也无法提供所有可用的元数据的键(key)参考列表。 `Field` 对象中保存的每个键可以由多个组件使用，并且只有这些组件知道这个键的存在。您可以根据自己的需求，定义使用其他的 `Field` 键。设置 `Field` 对象的主要目的就是在一个地方定义好所有的元数据。 一般来说，那些依赖某个字段的组件肯定使用了特定的键(key)。您必须查看组件相关的文档，查看其用了哪些元数据键(metadata key)。

需要注意的是，用来声明item的 `Field` 对象并没有被赋值为class的属性。 不过您可以通过 `Item.fields` 属性进行访问。

##与Item配合(Working with Items)##

接下来以 下边声明 的 <font color=red>`Product`</font> item来演示一些item的操作。您会发现API和 dict API 非常相似。

##创建Items(Creating items)##

	>>> product = Product(name='Desktop PC', price=1000)
	>>> print product
	Product(name='Desktop PC', price=1000)

##获取字段值(Getting field values)##

	>>> product['name']
	Desktop PC
	>>> product.get('name')
	Desktop PC
	
	>>> product['price']
	1000
	
	>>> product['last_updated']
	Traceback (most recent call last):
	    ...
	KeyError: 'last_updated'
	
	>>> product.get('last_updated', 'not set')
	not set
	
	>>> product['lala'] # getting unknown field
	Traceback (most recent call last):
	    ...
	KeyError: 'lala'
	
	>>> product.get('lala', 'unknown field')
	'unknown field'
	
	>>> 'name' in product  # is name field populated?
	True
	
	>>> 'last_updated' in product  # is last_updated populated?
	False
	
	>>> 'last_updated' in product.fields  # is last_updated a declared field?
	True
	
	>>> 'lala' in product.fields  # is lala a declared field?
	False

##设置字段值(Setting field values)##

	>>> product['last_updated'] = 'today'
	>>> product['last_updated']
	today
	
	>>> product['lala'] = 'test' # setting unknown field
	Traceback (most recent call last):
	    ...
	KeyError: 'Product does not support field: lala'

##获取所有获取到的值(Accessing all populated values)##

您可以使用 `dict API` 来获取所有的值：

	>>> product.keys()
	['price', 'name']
	
	>>> product.items()
	[('price', 1000), ('name', 'Desktop PC')]

##其他常用任务(Other common tasks)##

复制items
	
	>>> product2 = Product(product)
	>>> print product2
	Product(name='Desktop PC', price=1000)
	
	>>> product3 = product2.copy()
	>>> print product3
	Product(name='Desktop PC', price=1000)

根据item创建字典(dict):

	>>> dict(product) # create a dict from all populated values
	{'price': 1000, 'name': 'Desktop PC'}

根据字典(dict)创建item:
	
	>>> Product({'name': 'Laptop PC', 'price': 1500})
	Product(price=1500, name='Laptop PC')
	
	>>> Product({'name': 'Laptop PC', 'lala': 1500}) # warning: unknown field in dict
	Traceback (most recent call last):
	    ...
	KeyError: 'Product does not support field: lala'

##扩展Item(Extending Items)##

您可以通过继承原始的Item来扩展item(添加更多的字段或者修改某些字段的元数据)。

例如：

	class DiscountedProduct(Product):
	    discount_percent = scrapy.Field(serializer=str)
	    discount_expiration_date = scrapy.Field()

您也可以通过使用原字段的元数据,添加新的值或修改原来的值来扩展字段的元数据:
	
	class SpecificProduct(Product):
	    name = scrapy.Field(Product.fields['name'], serializer=my_serializer)

这段代码在保留所有原来的元数据值的情况下添加(或者覆盖)了 name 字段的 serializer 。

#Item对象(Item object)#

<table>
<tr>
<td>
<font color=green>class</font> scrapy.item.Item([arg])
</td>
</tr>
</table>

返回一个根据给定的参数可选初始化的item。

Item复制了标准的 dict API 。包括初始化函数也相同。Item唯一额外添加的属性是:
`fields`

一个包含了item所有声明的字段的字典，而不仅仅是获取到的字段。该字典的key是字段(field)的名字，值是 Item 声明中使用到的 `Field` 对象。

##Field对象(Field object)##

<table>
<tr>
<td>
<font color=green>class</font> scrapy.item.Field([arg])
</td>
</tr>
</table>

`Field` 仅仅是内置的 dict 类的一个别名，并没有提供额外的方法或者属性。换句话说， `Field` 对象完完全全就是Python字典(dict)。被用来基于类属性(class attribute)的方法来支持 item声明语法 。