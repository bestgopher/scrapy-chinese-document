# Spiders Contracts #

<font color=orange>
NOTE:</br>
这是一个新引入(Scrapy 0.15)的特性，在后续的功能/API更新中可能有所改变，查看 release notes 来了解更新。
</font>

测试spider是一件挺烦人的事情，尤其是只能编写单元测试(unit test)没有其他办法时，就更恼人了。 Scrapy通过合同(contract)的方式来提供了测试spider的集成方法。

您可以硬编码(hardcode)一个样例(sample)url， 设置多个条件来测试回调函数处理repsponse的结果，来测试spider的回调函数。 每个contract包含在文档字符串(docstring)里，以 <font color=red>`@`</font> 开头。 查看下面的例子:

	def parse(self, response):
	    """ This function parses a sample response. Some contracts are mingled
	    with this docstring.
	
	    @url http://www.amazon.com/s?field-keywords=selfish+gene
	    @returns items 1 16
	    @returns requests 0 0
	    @scrapes Title Author Year Price
	    """

该回调函数使用了三个内置的contract来测试:

<table><tr><td>
<font color=green>class</font> scrapy.contracts.default.UrlContract
</td></tr></table>

该constract(<font color=red>`@url`</font>)设置了用于检查spider的其他constract状态的样例url。 该contract是必须的，所有缺失该contract的回调函数在测试时将会被忽略:

	@url url

<table><tr><td>
<font color=green>class</font> scrapy.contracts.default.ReturnsContract
</td></tr></table>

该contract(<font color=red>`@returns`</font>)设置spider返回的items和requests的上界和下界。 上界是可选的:

	@returns item(s)|request(s) [min [max]]

<table><tr><td>
<font color=green>class</font> scrapy.contracts.default.ScrapesContract
</td></tr></table>

该contract(<font color=red>`@scrapes`</font>)检查回调函数返回的所有item是否有特定的fields:

	@scrapes field_1 field_2 ...

使用 `check` 命令来运行contract检查。

## 自定义Contracts(Custom Contracts) ##

如果您想要比内置scrapy contract更为强大的功能，可以在您的项目里创建并设置您自己的 contract，并使用 `SPIDER_CONTRACTS` 设置来加载:

	SPIDER_CONTRACTS = {
	    'myproject.contracts.ResponseCheck': 10,
	    'myproject.contracts.ItemValidate': 10,
	}

每个contract必须继承 `scrapy.contracts.Contract` 并覆盖下列三个方法:

<table><tr><td>
<font color=green>class</font> scrapy.contracts.Contract(method, *args)
</td></tr></table>

&nbsp;&nbsp;&nbsp;**参数**：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**method** (function) - contract所关联的回调函数

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**args**(list) - 传入docstring的(以空格区分的)argument列表(list)


<font color=green>adjust\_request\_args(args)</font></br>
接收一个 <font color=red>`字典(dict)`</font> 作为参数。该参数包含了所有 Request 对象 参数的默认值。该方法必须返回相同或修改过的字典。

<font color=green>pre_process(response)</font></br>
该函数在sample request接收到response后，传送给回调函数前被调用，运行测试。

<font color=green>post_process(output)</font></br>该函数处理回调函数的输出。迭代器(Iterators)在传输给该函数前会被列表化(listified)。


该样例contract在response接收时检查了是否有自定义header。 在失败时抛出 `scrapy.exceptions.ContractFaild` 来展现错误：

	from scrapy.contracts import Contract
	from scrapy.exceptions import ContractFail
	
	class HasHeaderContract(Contract):
	    """ Demo contract which checks the presence of a custom header
	        @has_header X-CustomHeader
	    """
	
	    name = 'has_header'
	
	    def pre_process(self, response):
	        for header in self.args:
	            if header not in response.headers:
	                raise ContractFail('X-CustomHeader not present')