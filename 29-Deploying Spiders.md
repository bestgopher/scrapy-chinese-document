#部署Spiders(Deploying Spiders)#

这一节介绍了部署你的Scrapy爬虫，定期运行它们的不同选项。在(早期)开发阶段在你本地机器上运行spider是非常方便的，但是当你需要执行长时间运行的spiders或者移动spider到生产环境运行时，本地机器就不够了。
这是解决部署spider的问题。

部署Scrapy Spiders的热门选择是：

  - Scrapyd(开源)
  - Scrapy Cloud(基于云)

##部署到Scrapyd服务器上(Deploying to a Scrapyd Server)##

Scrapyd是一个运行Scrapy Spider的开源的应用程序。它提供了具有HTTP API的服务器，能够运行和监控Scrapy spiders。

为了部署spiders到scrapyd上，你可以使用scrapyd-client包提供的scrapyd-deploy工具。请参考 [scrapyd-deploy documentation](https://scrapyd.readthedocs.io/en/latest/deploy.html) 获取更多的信息。

Scrapyd被一些Scrapy开发者维护着。

##部署到Scrapy Cloud上(Deploying to Scrapy Cloud)##

[Scrapy Cloud](https://scrapinghub.com/scrapy-cloud?_ga=2.149222832.723751010.1547428575-445645531.1543541242)是scrapy背后的公司Scrapinghub提供的基于云的托管服务。

Scrapy Cloud无需设置和监控服务器，提供了一个很棒的UI界面来管理spiders，查看已爬取的item，日志和统计信息。

为了部署spider到Scrapy Cloud上， 你可以使用[shub](https://doc.scrapinghub.com/shub.html?_ga=2.138139318.723751010.1547428575-445645531.1543541242)命令行工具。请参考 [Scrapy Cloud documentation](https://doc.scrapinghub.com/scrapy-cloud.html?_ga=2.242481156.723751010.1547428575-445645531.1543541242) 获取更多信息。

Scrapy Cloud与Scrapyd兼容，可以根据需要在它们之间切换 - 像 <font color=red>`scrapyd-deploy`</font>一样从<font color=red>`scrapy.cfg`</font>文件中读取配置。