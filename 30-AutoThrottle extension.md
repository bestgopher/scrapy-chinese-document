# 自动限速(AutoThrottle)扩展(AutoThrottle extension) #

该扩展能根据Scrapy服务器及您爬取的网站的负载自动限制爬取速度。

## 设计目的(Design goals) ##

  1. 更友好的对待网站，而不使用默认的下载间隔：<font color=red>`0`</font>。
  2. 自动调整scrapy来优化下载速度，使得用户不用调节下载间隔来找到优化的值。用户只需指定允许的最大并发请求数，剩下的都交给扩展来完成。

## 它怎样工作?(How it works) ##

AutoThrottle扩展调整动态地调整下载间隔，使得spider把发送的 `AUTOTHROTTLE_TARGET_CONCURRENCY`个并行请求平均到每个远程网站上。

它通过下载网页延迟来计算发送请求的间隔。主要思想如下：如果服务器需要 <font color=red>`latency`</font> 秒响应，客户端应该每 <font color=red>`latency/N`</font> 秒发送一个请求，这样就有 <font color=red>`N`</font> 个并行的请求处理。

使用`CONCURRENT_REQUESTS_PER_DOMAIN` 或者 `CONCURRENT_REQUESTS_PER_IP`选项可以设置一个小的固定间隔，对并发使用硬限制，来代替调整间隔。这个提供了与AutoThrottle相似的效果，但是也有些很重要的区别：

  - 因为下载间隔小，所以将会有请求的突然爆发(请求瞬间增多)。
  - 通常非200(错误)的响应比常规响应返回得更快，因此当服务器开始返回错误的时候，小的下载间隔和硬并行限制爬虫将更快的向服务器发送请求。但是这与爬虫应该做的完全相反-在错误的情况下，放慢爬取速度更有意义：这些错误有可能是因为请求频率太高引起的。

AutoThrottle没有这些问题。

## 限流算法(Throttling algorithm) ##

AutoThrottle算法调整下载延迟基于以下几个规则：

  1. spiders总是以 `AUTOTHROTTLE_START_DELAY` 下载间隔开始；
  2. 当收到一个响应，目标下载间隔(target download delay)通过 <font color=red>`latency/N`</font> 计算出来，这里的 <font color=red>`latency`</font> 是响应的时间，<font color=red>`N`</font> 是 `AUTOTHROTTLE_TARGET_CONCURRENCY` 设置的值；
  3. 下一个请求的下载间隔设置为先前的下载间隔和目标下载间隔(target download delay)的平均值。
  4. 非200响应码不允许减少间隔；
  5. 下载间隔不能小于 `DOWNLOAD_DELAY`或者大于 `AUTHROTTLE_MAX_DELAY`。

<font color=orange>
NOTE:</br>
AutoThrottle扩展尊重Scrapy并行和间隔的标准设置。这意味着它将尊重 `CONCURRENT_REQUESTS_PER_DOMAIN` 和 `CONCURRENT_REQUESTS_PER_IP` 选项，下载间隔不会设置低于 `DOWNLOAD_DELAY`。
</font>

在Scrapy中，下载等待时间是建议TCP链接到接受到HTTP响应头的时间。

记住这个延迟时间在一个合作多任务的环境中非常难精确地测量，因为Scrapy忙于处理Spider的回调，例如，不能参与下载。然而，这些延迟时间仍然合理地估计Scrapy(最终估算服务器)的繁忙程度，这个扩展也是在这个前提上创建的。

## 设置(Settings) ##

这些设置用来控制AutoThrottle：
 
  - `AUTOTHROTTLE_ENABLED`
  - `AUTOTHROTTLE_START_DELAY`
  - `AUTOTHROTTLE_MAX_DELAY`
  - `AUTOTHROTTLE_TARGET_CONCURRENCY`
  - `AUTOTHROTTLE_DEBUG`
  - `CONCURRENT_REQUESTS_PER_DOMAIN`
  - `CONCURRENT_REQUESTS_PER_IP`
  - `DOWNLOAD_DELAY`

## AUTOTHROTTLE_ENABLED ##
默认：<font color=red>`False`</font>



## AUTOTHROTTLE_START_DELAY ##
默认：<font color=red>`5.0`</font>

初始的下载延迟(秒)

启用AutoThrottle扩展。

## AUTOTHROTTLE_MAX_DELAY ##
默认：<font color=red>`60.0`</font>

在搞延迟下设置的最大的下载延迟(秒)

## AUTOTHROTTLE\_TARGET\_CONCURRENCY ##
默认：<font color=red>`1.0`</font>

Scrapy并行向远程网站发送请求的平均数。

默认情况下，AutoThrottle调整一个并发请求发送给远程网站的间隔。设置这个选项一个更高的值(例如<font color=red>`2.0`</font>)增加服务器的吞吐量和负载。一个更低的 <font color=red>`AUTOTHROTTLE_TARGET_CONCURRENCY`</font>(例如<font color=red>`0.5`</font>)让爬虫更加保守和有礼貌。

记住当AutoThrottle扩展启用了， `CONCURRENT_REQUESTS_PER_DOMAIN` 和 `CONCURRENT_REQUESTS_PER_IP` 选项仍然被遵守。这就意味如果 <font color=red>`AUTOTHROTTLE_TARGET_CONCURRENCY`</font>设置的值比
`CONCURRENT_REQUESTS_PER_DOMAIN` 或者`CONCURRENT_REQUESTS_PER_IP`的值高，爬虫将不会达到这个数量的请求并法数。

## AUTOTHROTTLE_DEBUG ##
默认：<font color=red>`False`</font>

起用AutoThrottle调试(debug)模式，展示每个接收到的response。 您可以通过此来查看限速参数是如何实时被调整的。

