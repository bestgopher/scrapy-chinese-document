#使用Firebug进行爬取(Using Firebug for scraping)#

<font color=orange>
NOTE:</br>
本教程所使用的样例站Google Directory已经 被Google关闭 了。不过教程中的概念任然适用。 如果您打算使用一个新的网站来更新本教程，您的贡献是再欢迎不过了。 详细信息请参考 Contributing to Scrapy 。
</font>

##介绍(Introduction)##

本文档介绍了如何使用 Firebug (一个Firefox的插件)来使得爬取更为简单，有趣。 更多有意思的Firefox插件请参考 对爬取有帮助的实用Firefox插件 。 使用Firefox插件检查页面需要有些注意事项: 在浏览器中检查DOM的注意事项 。

在本样例中将展现如何使用 Firebug 从 Google Directory 来爬取数据。 Google Directory 包含了 入门教程 里所使用的 Open Directory Project 中一样的数据，不过有着不同的结构。

Firebug提供了非常实用的 检查元素 功能。该功能允许您将鼠标悬浮在不同的页面元素上， 显示相应元素的HTML代码。否则，您只能十分痛苦的在HTML的body中手动搜索标签。

在下列截图中，您将看到 检查元素 的执行效果。
