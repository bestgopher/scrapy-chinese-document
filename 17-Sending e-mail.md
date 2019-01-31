#Sending e-mail#

虽然Python通过 smtplib 库使得发送email变得很简单，Scrapy仍然提供了自己的实现。 该功能十分易用，同时由于采用了 Twisted非阻塞式(non-blocking)IO ，其避免了对爬虫的非阻塞式IO的影响。 另外，其也提供了简单的API来发送附件。 通过一些 settings 设置，您可以很简单的进行配置。

#快速示例(Quick example)#

有两种方法可以创建邮件发送器(mail sender)。 您可以通过标准构造器(constructor)创建:

	from scrapy.mail import MailSender
	mailer = MailSender()

或者您可以传递一个Scrapy设置对象，其会参考 settings:

	mailer = MailSender.from_settings(settings)

这是如何来发送邮件了(不包括附件):

	mailer.send(to=["someone@example.com"], subject="Some subject", body="Some body", cc=["another@example.com"])


##MailSender类参考手册(MailSender class reference)##

在Scrapy中发送email推荐使用MailSender。其同框架中其他的部分一样，使用了 Twisted非阻塞式(non-blocking)IO 。

<table><tr><td>
<font color=green>scrapy</font>  &nbsp;scrapy.mail.MailSender(smtphost=None, mailfrom=None, smtpuser=None, smtppass=None, smtpport=None)

</td></tr></table>

**参数**：

  - **smtphost** (str) - 发送email的SMTP主机(host)。如果省略了，则使用 `MAIL_HOST`。
  - **mailfrom** (str) - 用于发送email的地址(address)(填写在 <font color=red>`From:`</font>标题) 。 如果省略了，则使用 `MAIL_FROM` 。
  - **smtpuser** - SMTP用户。如果省略了,则使用 `MAIL_USER` 。 如果未给定，则将不会进行SMTP认证(authentication)。
  - **smtppass** (str) -  SMTP认证的密码
  - **smtpport** (int) - SMTP连接的端口
  - **smtptls** (boolean) - 强制使用STARTTLS
  - **smtpssl** (boolean) - 强制使用SSL连接

**classmethod from_settings(settings)**</br>
使用Scrapy设置对象来初始化对象。其会参考 这些Scrapy设置.</br>
**参数**：settings (scrapy.settings.Settings object) – the e-mail recipients

**send(to, subject, body, cc=None, attachs=(), mimetype='text/plain')**</br>
发送email到给定的接收者。</br>
**参数**：

  - **to** (list) – email接收者
  - **subject** (str) – email主题
  - **cc**(list) – 抄送的人
  - **body** (str) – email的内容
  - **attachs** (iterable) – 附件。可迭代的元组 <font color=red>`(attach_name, mimetype,  file_object)`</font>。 </font color=red>`attach_name`</font> 是一个在email的附件中显示的名字的字符串， <font color=red>`mimetype`</font> 是附件的mime类型， <font color=red>`file_object`</font> 是包含附件内容的可读的文件对象。
  - **mimetype** (str) – email的mime类型
  - **charset** (str) - 邮件文本使用的字符编码格式

#Mail settings#

这些设置定义了 `MailSender` 构造器的默认值。其使得在您不编写任何一行代码的情况下，为您的项目配置实现email通知的功能。

##MAIL_FROM##

默认值: <font color=red>`'scrapy@localhost'`</font>

用于发送email的地址

## MAIL_HOST ##

默认值: <font color=red>`'localhost'`</font>

发送email的SMTP主机

##MAIL_PORT##

默认值: <font color=red>`25`</font>

发用邮件的SMTP端口。

##MAIL_USER##

默认值: <font color=red>`None`</font>

SMTP用户。如果未给定，则将不会进行SMTP认证(authentication)。

##MAIL_PASS##

默认值: <font color=red>`None`</font>

用于SMTP认证，与 MAIL_USER 配套的密码。

##MAIL_TLS##

默认值: <font color=red>`False`</font>

强制使用STARTTLS。STARTTLS能使得在已经存在的不安全连接上，通过使用SSL/TLS来实现安全连接。

##MAIL_SSL##

默认值: <font color=red>`False`</font>

强制使用SSL加密连接。