# 下载和处理文件和图片(Downloading and processing files and images) #

Scrapy提供了一个 item pipeline ，来下载属于某个特定项目的文件(比如，当你抓取产品时，也想把它们的图片下载到本地)。这些pipelines共享一些功能和结构(我们把它们称作media pipelines)，你通常使用 Files Pipeline 或者 Images Pipeline。

两个管道实现了这些功能：

  - 避免重复下载刚下载过的文件
  - 指定存储文件的位置(文件系统，Amazon S3，Google云)

Images Pipeline有一些额外的功能来处理图片：

  - 将所有下载的图片转化为常用的格式(JPG)和模式(RGB)
  - 生成缩略图
  - 检测图像的宽/高，确保它们满足最小限制

这个管道也会为那些当前安排好要下载的媒体保留一个内部队列，并将那些到达的包含相同媒体的项目连接到那个队列中。 这可以避免多次下载几个项目共享的同一个媒体。

## 使用Files Pipeline(Using the Files Pipeline) ##

当使用 `FilesPipeline` ，典型的工作流程如下所示:

  1. 在一个爬虫里，你抓取一个item，把其中期望的URL放入 <font color=red>`files_urls`</font> 字段内(这个字段需要一个列表)。
  2. 项目从爬虫内返回，进入项目管道。
  3. 当项目进入 `FilesPipeline`，<font color=red>`files_urls`</font> 组内的URLs将被Scrapy的调度器和下载器（这意味着调度器和下载器的中间件可以复用）安排下载，当优先级更高，会在其他页面被抓取前处理。项目会在这个特定的管道阶段保持“locker”的状态，直到完成文件的下载（或者由于某些原因未完成下载）。
  4. 当文件下载完，item另一个字段(<font color=red>`files`</font>)将被更新到结构中。这个字段将包含一个字典列表，其中包括下载文件的信息，比如下载路径、源抓取地址（从 <font color=red>`file_urls`</font> 组获得）和文件的校验码。 <font color=red>`files`</font> 列表中的文件顺序将和源 <font color=red>`file_urls`</font> 字段保持一致。如果某个文件下载失败，将会记录下错误信息，文件也不会出现在 <font color=red>`files`</font> 字段中

## 使用Images Pipeline(Using the Images Pipelines) ##

使用 `ImagesPipeline` 与使用 `FilesPipeline` 很多相同点，除了使用的字段名不同：item的<font color=red>`image_urls`</font>字段用来存储图片的URLs，<font color=red>`images`</font>字段存储下载图片的信息。

使用`ImagesPipeline`下载图片文件的优点是你可以配置一些额外的功能，像生成缩略图和通过尺寸过滤图片。

Images Pipeline使用 [Pillow](https://github.com/python-pillow/Pillow) 库来生成缩略图和转化图片为JPEG/RGB格式的，因此你需要安装这个库。 [Python Imaging Library](http://www.pythonware.com/products/pil/) (PIL) 在很多情况下也能工作，但是做所周知的是在安装的时候会有许多问题，因此我们建议使用 Pillow 而不是 PIL。

## 开启你的媒体管道(Enabling your Media Pipeline) ##

为了开启你的图片管道，你首先需要在项目中添加它 `ITEM_PIPELINES `setting。

对于Images Pipeline来说，使用：

	ITEM_PIPELINES = {'scrapy.pipelines.images.ImagesPipeline': 1}

对于Files Pipeline来说，使用：

	ITEM_PIPELINES = {'scrapy.pipelines.files.FilesPipeline': 1}

注意：你也可以同时使用Files和Images管道。

然后，将目标存储设置为一个可以的值，这个将用来存储下载下来的文件。不配置存储目录pipeline仍然不能使用，即使你配置了`ITEM_PIPELINES`。

对于Files Pipeline，设置`FILES_STORE`：

	FILES_STORE = '/path/to/valid/dir'

对于Images Pipeline，设置`IMAGES_STORE`：

	IMAGES_STORE = '/path/to/valid/dir'

## 支持的存储方式(Supported Storage) ##

文件系统是目前官方唯一支持的存储方式，但是也支持在Amazon S3和Google云上存储文件。

### 文件系统存储(File system storage) ###

文件使用 URLs的 [SHA1 hash](https://en.wikipedia.org/wiki/Secure_Hash_Algorithms)值作为文件名来保存的。

例如，下面的图片url：

	http://www.example.com/image.jpg

它的SHA1 hash值为：
	
	3afec3b4765f8f0a07b78f98c07b83f013567a0a

将会下载和存储为下面的文件：

	<IMAGES_STORE>/full/3afec3b4765f8f0a07b78f98c07b83f013567a0a.jpg

这里：

  - <font color=red>`<IMAGES_STORE>`</font> 是为 Images Pipeline定义的目录，值为 `IMAGES_STORE` 设置定义的值。
  - <font color=red>`full`</font> 是一个子目录。用于将完整图片和缩略图(如果使用)分开。

### Amazon s3 ###

`FILES_STORE`和`IMAGES_STORE`可以为Amazon s3存储平台。Scrapy能够自动上传文件到这个存储平台。

举例，这是一个有效的 `IMAGES_STORE`值：

	IMAGES_STORE = 's3://bucket/images'

你可以修改用于存储文件的访问控制列表(Access Control List，ACL)策略，它通过 `FILES_STORE_S3_ACL` 和 `IMAGES_STORE_S3_ACL`设置定义。默认情况下，ACL设置为 <font color=red>`private`</font>。要使文件公开可用，使用 <font color=red>`pulic-red`</font>策略。

	IMAGES_STORE_S3_ACL = 'public-read'

### Google云存储(Google Cloud Storage) ###

`FILES_STORE`和`IMAGES_STORE`可以为Google云存储平台。Scrapy能够自动上传文件到这个存储平台。


举例，这是一个有效的 `IMAGES_STORE` 和 `GCS_PROJECT_ID` 的值：

	IMAGES_STORE = 'gs://bucket/images/'
	GCS_PROJECT_ID = 'project_id'

## 使用举例(Usage example) ##

为了使用媒体pipeline，首先开启它。

然后，如果Spider返回一个由URLs键的字典(<font color=red>`file_urls`</font>或者<font color=red>`images_urls`</font>分别对应Files Pipeline和Images Pipeline)，pipeline将会把结果放在各自的键上(<font color=red>`files`</font>或者<font color=red>`images`</font>)。

如果你更喜欢使用 `Item`，定义一个有着必要字段的item，像这个为Images Pipeline举的例子一样：

	import scrapy

	class MyItem(scrapy.Item):
	
	    # ... other item fields ...
	    image_urls = scrapy.Field()
	    images = scrapy.Field()

如果你想为URLs键或者结果键使用其他的字段名，也可以重写它。

对于Files Pipeline来说，设置 `FILES_URLS_FIELD`和/或`FILES_RESULT_FIELD`：
	
	FILES_URLS_FIELD = 'field_name_for_your_files_urls'
	FILES_RESULT_FIELD = 'field_name_for_your_processed_files'

对于Images Pipeline来说，设置 `IMAGES_URLS_FIELD`和/或`IMAGES_RESULT_FIELD`：

	IMAGES_URLS_FIELD = 'field_name_for_your_images_urls'
	IMAGES_RESULT_FIELD = 'field_name_for_your_processed_images'

如果你需要一些更加复杂的东西，想要重写一个定义好的pipeline行为，可以看Extending the Media Pipelines。

如果你有许多继承自ImagePipeline的image pipeline，并且你想要不同的pipelines有着不同的设置。你可以设置键以pipeline类的大写类名开头。例如，如果你的pipeline名为MyPipeline，而且你想要定义IMAGES_URLS_FIELD，你可以定义MYPIPELINE_IMAGES_URLS_FIELD，这个设置将会作为使用的设置。

## 额外的特性(Additional features ##

### 文件到期(File expiration) ###

File Pipeline避免下载最近已经下载的图片。使用 `FILES_EXPIRES`(Image Pipeline情况下使用`IMAGES_EXPIRES`) 设置可以调整失效期限，可以用天数来指定:

	# 120 days of delay for files expiration
	FILES_EXPIRES = 120
	
	# 30 days of delay for images expiration
	IMAGES_EXPIRES = 30

这两个值的默认值都为90天。

如果你有FilesPipeline的子类，你想为它设置不同的值，你可以设置以这个类名的大写开头的键，例如，如果你的pipeline名为MyPipeline，你可以这样设置：

	MYPIPELINE_FILES_EXPIRES = 180

MyPipeline将会设置180为生存时间。

### 图片生成缩略图(Thumbnail generation for images) ###

图片管道可以自动创建下载图片的缩略图。

为了使用这个特性，你需要设置 `IMAGES_THUMBS` 字典，其关键字为缩略图名字，值为它们的大小尺寸。

比如:

	IMAGES_THUMBS = {
	    'small': (50, 50),
	    'big': (270, 270),
	}

当你使用这个特性时，图片管道将使用下面的格式来创建各个特定尺寸的缩略图:

	<IMAGES_STORE>/thumbs/<size_name>/<image_id>.jpg

其中:

  - <font color=red>`<size_name>`</font> 是 `IMAGES_THUMBS` 字典键（<font color=red>`small`</font>， <font color=red>`big`</font> ，等）
  - <font color=red>`<image_id>`</font> 是图像url的 SHA1 hash

例如使用 <font color=red>`small`</font> 和 <font color=red>`big`</font> 缩略图名字的图片文件:

	<IMAGES_STORE>/full/63bbfea82b8880ed33cdb762aa11fab722a90a24.jpg
	<IMAGES_STORE>/thumbs/small/63bbfea82b8880ed33cdb762aa11fab722a90a24.jpg
	<IMAGES_STORE>/thumbs/big/63bbfea82b8880ed33cdb762aa11fab722a90a24.jpg

第一个是从网站下载的完整图片。

### 滤出小图片(Filtering out small images) ###

当使用Images Pipeline时，你可以丢掉那些过小的图片，只需在`IMAGES_MIN_HEIGHT` 和 `IMAGES_MIN_WIDTH` 设置中指定最小允许的尺寸。

比如：

	IMAGES_MIN_HEIGHT = 110
	IMAGES_MIN_WIDTH = 110

注意：这些尺寸一点也不影响缩略图的生成。

设置一个或者两个约束都行。当两个都设置时，只有满足两个最小尺寸的图片才能被保存。如上面的设置的话，(105 x 105)、(105 x 200)、(200 x 105)的图片都会被丢弃，因为至少有一维比约束短。

默认情况下，没有尺寸限制，因此所有图片都将处理。

### 允许重定向(Allowing redirections) ###

默认情况下媒体pipelines忽略重定向，即HTTP重定向到媒体文件URL请求将意味着媒体下载被视为失败。

要处理媒体重定向，请将此设置设置为 <font color=red>`True`</font>：

	MEDIA_ALLOW_REDIRECTS = True

## 扩展媒体pipelines(Extending the Media Pipelines) ##

下面是你可以在定制的图片管道里重写的方法：

<table><tr><td>
<font color=green>class</font> scrapy.pipelines.files.FilesPipeline
</td></tr></table>

&nbsp;&nbsp;&nbsp;&nbsp;**get\_media\_requests(item, info)**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在工作流程中    可以看到，管道会得到文件的URL并从项目中下载。为了这么做，你需要重写 `get_media_requests()` 方法，并对各个文件URL返回一个`Request`:

	def get_media_requests(self, item, info):
	    for file_url in item['file_urls']:
	        yield scrapy.Request(file_url)

这些请求将被管道处理，当它们完成下载后，结果将以2个元素的元组组成的列表形式传送到 `item_completed()` 方法。每个元组包含<font color=red>`(success, file_info_or_error)`</font>：

  - <font color=red>`success`</font> 是一个布尔值，当图片成功下载时为 <font color=red>`True`</font> ，因为某个原因下载失败为 <font color=red>`False`</font>。
  - <font color=red>`file_info_or_error`</font> 是一个包含下列关键字的字典（如果成功为 <font color=red>`True`</font>）或者出问题时为 Twisted Failure 。
 
    1.  <font color=red>`url`</font> - 文件下载的url。这是从 `get_media_requests()` 方法返回请求的url。
    2.  <font color=red>`path`</font> - 文件存储的路径（相对于 `FILES_STORE`）。
    3.  <font color=red>`checksum`</font> - 文件内容的 MD5 hash

`item_completed()` 接收的元组列表需要保证与 `get_media_requests()` 方法返回请求的顺序相一致。下面是 <font color=red>`results`</font> 参数的一个典型值：

	[(True,
	  {'checksum': '2b00042f7481c7b056c4b410d28f33cf',
	   'path': 'full/0a79c461a4062ac383dc4fade7bc09f1384a3910.jpg',
	   'url': 'http://www.example.com/files/product1.pdf'}),
	 (False,
	  Failure(...))]

默认 `get_media_requests()` 方法返回 <font color=red>`None`</font> ，这意味着项目中没有文件可下载。


&nbsp;&nbsp;&nbsp;&nbsp;**item_completed(results, item, info)**

当一个单独项目中的所有文件请求完成时（要么完成下载，要么因为某种原因下载失败）， FilesPipeline.item_completed() 方法将被调用。

item_completed() 方法需要返回一个输出，其将被送到随后的项目管道阶段，因此你需要返回（或者丢弃）item，如你在任意管道里所做的一样。

这里是一个 item_completed() 方法的例子，其中我们将下载的文件路径（传入到results中）存储到 <font color=red>`file_paths`</font> 项目组中，如果其中没有文件，我们将丢弃项目:

	from scrapy.exceptions import DropItem
	
	def item_completed(self, results, item, info):
	    file_paths = [x['path'] for ok, x in results if ok]
	    if not file_paths:
	        raise DropItem("Item contains no files")
	    item['file_paths'] = file_paths
	    return item

默认情况下， `item_completed()` 方法返回item。


<table><tr><td>
<font color=green>class</font> scrapy.pipelines.images.ImagesPipeline
</td></tr></table>

`ImagesPipeline` 是 `FilesPipeline` 的扩展，定制了文件名和为图片添加了自定义的行为。

&nbsp;&nbsp;&nbsp;&nbsp;**get_media_requests(item, info)**

工作的方式与 `FilesPipeline.get_media_requests()`方法一样，但是图片的urls使用不同的item字段名。

必须返回每个图片URL的Request。

&nbsp;&nbsp;&nbsp;&nbsp;**item_completed(results, item, info)**

`ImagesPipeline.item.item_completed()` 方法在当每个item中的所有image 请求都完成时调用(要么完成下载，要么因为某种原因下载失败)。

工作的方式与 `FilesPipeline.item_completed()`方法一样，但是图片的下载结果使用不同的item字段名。

默认情况下， `item_completed()` 方法返回item。

## 定制图片管道的例子(Custom Images pipeline example) ##

下面是一个图片管道的完整例子，其方法如上所示：

	import scrapy
	from scrapy.contrib.pipeline.images import ImagesPipeline
	from scrapy.exceptions import DropItem
	
	class MyImagesPipeline(ImagesPipeline):
	
	    def get_media_requests(self, item, info):
	        for image_url in item['image_urls']:
	            yield scrapy.Request(image_url)
	
	    def item_completed(self, results, item, info):
	        image_paths = [x['path'] for ok, x in results if ok]
	        if not image_paths:
	            raise DropItem("Item contains no images")
	        item['image_paths'] = image_paths
	        return item