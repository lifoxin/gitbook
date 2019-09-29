**创建 project**

scrapy startproject example   

**创建项目quotes**

cd example

scrapy genspider quotes quotes.com

**执行 scrapy**

scrapy crawl quotes -o quotes.{jl,json,csv}  --nolog

**交互scrapy**

scrapy shell http://quotes.toscrape.com

resquest = scrapy.Request(url)

fetch(request)

response.xpath('//a/@href').extract()

**xpath选择器**

//div[@class="info"]/span/text()

//div[@class="info"]//a/@href

//*[contains(@class,"info") and contains(@class,info2)]//h1//text()
 
**css选择器** 

|子节|意义|
|-|-|
|.intro         | 代表class="intro"|
|.intro .intro1 | 代表class 下的 class|
|#id            | 代表 id="id"|
|p              | 代表p|
|div,p          | 代表所有div和p|
|div p          | 代表div 下的 p|
|a::attr(title) | 代表a标签下的title标签|
|a::text        | 代表a标签下的所有文本|

**xpath、css、re 串联使用**

response.css('#id').xpath('text()').re('[.0-9]+')

**pipeline 文件**

```python
import scrapy
import MySQLdb

class ValidateItem:    #去重有效数据
	def process_item(self,item,spider):
		for key in item.fields.keys():
			if key not in item:
				print('item dropped')
				raise scrapy.exceptions.DropItem('key %s is missing' % key)
			val = item[key]
			if not isinstance(val,str):
				print('item dropped')
				raise scrapy.exceptions.DropItem('value %s is not str' %val)
		return item

class TextWriterPipeline:  #写入文本
	def open_spider(self,spider):
		self.file = open('items.txt','w')
	def close_spider(self,spider):
		self.file.close()
	def process_item(self,item,spider):
		for k in ['text','author','tags']:
			value = item[k]
			self.file.write('%s: %s\n' % (k,value)
		self.file.write('\n')
		return item
		
class MysqlPipeline:  #写入数据库
	@classmethod     #实例化类，通过crawler引擎调用setting
	def from_crawler(cls,crawler):
		user = crawler.settings.get('USER')
		password = crawler.settings.get('PASSWORD')
		database = crawler.settings.get('DATABASE')
		charset = crawler.settings.get('CHARSET')
		dbtable = crawler.settings.get('DBTABLE')
		return cls(user,password,database,charset,dbtable)
	def __init__(self,user,password,database,charset,dbtable):
		self.user = user
		self.password = password
		self.database = database
		self.charset = charset
		self.dbtable = dbtable
	def open_spider(self,spider):
		self.conn = MySQLdb.connect(user='self.user',passwd='self.password',
									database='self.database',charset='self.utf8')
		self.cursor = self.conn.cursor()
	def close_spider(self,spider):
		self.conn.commit()
		self.conn.close()
	def process_item(self,item,spider):
		sql = 'insert into quotes values (%s,%s,%s)'
		data = (item['text',item['author'],item['tags']]
		self.cursor.execute(sql,args=data)
		return item
```

**MiddleWare 文件**

```python
import random
from scrapy.downloadermiddlewares.useragent import UserAgentMiddleware

class FilterOutUrl:  #过滤掉不是以page开头的url
	def process_spider_output(self,response,result,spider):
		for i in result:
			path ='/' + '/'.join(i.url.split('/')[3:])
			# 如果 allowed_paths 不存在，或者None,就不过滤
			if getattr(spider,'allowed_paths',None)
				if isinstance(i,scrapy.Request):
					if not path.startwith('/page/'):
						continue
			yield i

class RandomUserAgentMiddleware(UserAgentMiddleware):  #下载的时候使用随机的user-agent
	@classmethod
	def from_crawler(cls, crawler):
		return cls(crawler.settings.get('USER_AGENT_LIST',None))
	def __init__(self, user_agent_list):
		if user_agent_list is None:
			self.agents = ['scrapy']
		else:
			self.agents = open(user_agent_list).read().splitlines()
	def process_request(self, request, spider):
		agent = random.choice(self.agents)
		request.headers["User-Agent"] = agent

class ProxyMiddleware:  #下载启用代理ip
	def process_request(self,request, spider):
		request.meta['proxy'] = "http://39.134.10.21:8080"
```		

**setting 文件**

```python
ITEM_PIPELINES = {
	‘scrapy.pipelines.images.ImagesPipeline’:1,    	 #启用图片管道
	'quotes.pipelines.ValidateItem':300,
	'quotes.pipelines.TextWriterPipeline':400,
}
SPIDER_MIDDLEWARES = {
	'quotes.middlewares.FilterOutUrl':10000          #启用中间件
}
DOWNLOADER_MIDDLEWARES = {                      	 #下载中间件
	'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware':500,
	'prox.middlewares.ProxyMiddleware':100
}
IMAGES_STORE = '/tmp/images'         				 #图片存储的地方
ROBOTSTXT_OBEY = False                               #不遵守robot
USER = 'scrapy'
PASSWORD = 'scrapy'
DATABASE = 'scrapy'
CHARSET = 'utf8'
DATABLE = 'quotes'
USER_AGENT_LIST = '/tmp/ua_list.txt'
```
**item 文件**

```python
from scrapy.item imort Item, Field
from scrapy.loader import ItemLoader
from scrapy.loader.processors import MapCompose, TakeFirst, Identity

def drop_symbol(text):
	return text[1:]
def urljoin(url,loader_context):
	response = loader_context['response']
	return response.urljoin(url)
class QuotesItem(Item):
	text = scrapy.Field()
	author = scrapy.Field()
	tags = scrapy.Field()
	images = scrapy.Field()                       #下载图片存放的地址
	images_urls = scrapy.Field()   				  #下载图片必须有的
	
class QuotesItemLoader(ItemLoader):                #如果scrapy使用loader,所以这里要定义loader
	default_item_class = QuotesItem
	default_output_processor = TakeFirst()         #第一个有效值
	tags_in = MapCompose(lambda x: x.split(','))   #列表
	tags_out = Identity()  						   #原样输出
	image_urls_in = MapCompose(urljoin)
	image_urls_out = Identity()                    #原样输出
	title_in = MapCompose(str.strip)
	price_in = MapCompose(drop_symbol,float)       #对爬取的数据格式化输入item
```	
	
**scrapy项目文件**

```python
import scrapy
from quotes.items import QuotesItem，QuotesItemLoader

from scrapy.shell import inspect_response  #调试模块，在代码运行的时候
inspect_response(response,self)

class proxSpider(scrapy.Spider):  #在运行程序中加参数
	name = 'prox'
	start_urls = ['http://www.gnu.org/']
	def start_reqests(self):
		target = getattr(self, 'target', self.start_urls[0])
		print('yy: ',self,yy, 'xx：',self,xx)
		yield scrapy.Request(target)
	def parse(self, response):
		print('processing page %s ' %response.url)
		
运行：
scrapy crawl prox -a target='https://www.wikipedia.org/' -a yy=1 -a xx=1

class DjangoSpider(scrapy.Spider):   #使用scrapy.FormRequest模拟登陆爬取
	name = 'django'
	def start_requests(self):
		login_url = "http://localhost:8000/polls/login/"
		yield scrapy.FormRequest(
			url = login_url,
			formdata={'username':'kyo','password':'abc'},
			callback=self.after_port
		)
	def after_port(self,response):
		if ' login failed' not in response.text:
			self.log('login successful')

class DjangoSpider(scrapy.Spider): 
#使用FormRequest.from_response模拟登陆表单有隐藏字段,登陆后，跳转questions_list页面，再爬取问题
	name = 'django'
	start_urls = ['http://127.0.0.1:8000/amin/login/']
	question_list_url = 'http://127.0.0.1:8000/admin/polls/question/'
	def parse(self,response):
		yield FormRequest.from_response(
			response,
			url = response.url,
			formdata = {'username':'admin','password':'abcd/1234'}
			callback=self.after_login
			)
	def after_login(self,response):
		if 'Log in' not in response.css('title::text').extract_first():
			yield scrapy.Request(
				self.question_list_url,
				callback=self.parse_question_list
				)
	def parse_question_list(self,response):  #urljoin添加href标签生成完整url
		for href in response.xpath('//tr//a/@href').extract():  
			href = response.urljoin(href)    
			yield scrapy.Request(href,callback=self.extract_data)
		for a in response.xpath('//tr//a'):  #follow模块根据a标签跟踪生成完整url,返回request
			yield response.follow(a, callback=self.extract_data)
	def extract_data(self,response):
		text = response.xpath('//input[@name="question_text"]/@value').extract_first()
		date = response.xpath('//input[@name="pub_date_0"]/@value').extract_first()
		time = response.xpath('//input[@name="pub_date_1"]/@value').extract_first()
		yield dict(text=text,date=date,time=time)

class MwSpider(scrapy.Spider):
	name = 'mw'
	start_urls = ['http://quotes.toscrape.com']
	allowed_domains = ['quotes.toscrapy.com']
	allowed_paths = ['/page/','/tag/']
	def parse(self,response):     #跟踪出所有a标签下的url
		print('parsing page %s' % response.url)
		for a in response.xpath('//a')
			yield response.follow(a)

class BooksSpider(scrapy.Spider):   #不使用 item loader 情况
	name = "books2"
	allowed_domain = ['books.toscrapy.com']
	start_urls = ['http://books.toscrape.com']
	def parse(self,response):  
		for li in response.xpath('//ol/li'):
			title = li.xpath('.//h3/a/text()').extract_first()
			price = li.css('.price_color::text').extract_first()
			price = float(price[1:])
			image_url = li.css('img::attr(src)').extract_first()
			image_url = response.ruljoin(image_url)
			image_urls = [image_url]
			yield dict(title = title,price=price,image_urls=image_urls)

class BasicSpider(scrapy.Spider):
	name = "basic"
	allow_domains = ["quotes.com"]
	start_urls = [
		"http://quotes.toscrape.com"
	]
	#使用crawlSpider 实现双向爬取，继承CrawlSpider,而不是Spider
	pattern1 = restrict_xpaths='//*[contains(@class,"text")]'
    pagelink1 = LinkExtractor(pattern1)
	pattern2 = restrict_xpaths='//*[@itemprop="url"]'
    pagelink2 = LinkExtractor(pattern2)
	# 可以写多个rule规则
    rules = [
        Rule(pagelink1),
		Rule(pagelink1, callback='parse_item'),
    ]
	def parse(self,response):
		#self.log("images:%s" % response.xpath('//img/@src').extract())
		next_selector = response.xpath('//*[contains(@class,"next")]//@href')
		for url in next_selector.extract():
			yield scrapy.requests(response.urljson(url))
		
		item_selector = response.xpath('//*[@itemprop="url"]/@href')
		for url in item_selector.extract():
			yield scrapy.requests(response.urljson(url),callback=self.parse_item)
	def parse_item(self,url):  #使用 item loader
		for quote in response.xpath('//div[@class="quote"]'):
			l = QuotesItemLoader(selector=quote,response=response)
			l.add_xpath('text','.//span[@class="text"]/text()')
			l.add_xpath('author','.//small[@class="author"]/text()')
			l.add_xpath('tags','.//meta[@class="keywords"]/text()')
			yield l.load_item()
			
class StackSpider(scrapy.Spider):  #从多网页爬取信息,组合成一个item
	name = 'stack'
	start_urls = ['https://stackoverflow.com/?tab=month']
	def parse(self, response):
		for div in response.css('div.question-summary'):
			title = div.css('div.summary h3 a::text').extract_first()
			tags = div.css('div.summary .tags a ::text').extract()
			mtime = div.css('div.summary .relativetime::attr(title)').extract_first()
			many = div.css('.cp div.mini-counts span::attr(title)').extract()
			vite, answer,view = [int(x.split()[]) for x in many]
			item = dict(title=title,tags=tags,mtime=mtime,vote=vote,
						answer=answer,view=view)
			anchor = div.css('div.summary h3 a')[0]
			yield response.follow(anchor,meta={'item':item},callback=self.get_dateil)
	def get_detail(self,response):
		item = response.meta['item']
		texts = response.css('div.post-text')[0].css('*:text').extract()
		detail = ''.join(texts)
		item['detail'] = detail 
		yield item
		
class UaSpiderSlow(scrapy.Spider):  #慢慢爬同一个网页
	name = 'slow'
	def start_requests(self):
		for i in range(2000):
			print('yidlding %s request' %i)
			yield Request('http://quotes.toscrape.com/page/1/?round=%s' %i,
							meta={'i':i})
	def parse(self, response):
		print('processing page %s'%response.meta['i'])
			
setting: 
	DOWNLOAD_DELAY = 1  #下载延迟 1*0.5s~1*1.5s 之间，模块不延迟

#通过telnet console控制爬取状态:telnet localhost 6023	
	est()  				爬取状态
	engine.pause()  	暂停
	enginge.paused()    暂停状态
	enginge.unpause(）  不暂停
	engine.stop()       停止
			
#存储爬虫任务信息。time 显示运行时间
	mkdir jobs
	time scrapy crawl slow -s JOBDIR=jobs -s DOWNLOAD_DELAY=1 --nolog

#通过Splash容器渲染JS网页爬取数据，安装docker和scrapy-splash模块
	docker pull scrapinghub/splash
	docker run -d --name splash -p 8050:8050 scrapinghub/splash
	pip install scrapy-splash

#配置setting默认项
	SPLASH_URL = 'http://localhost:8050'
	DOWNLOADER_MIDDLEWARES = {
		'scrapy_splash.SplashCookiesMiddleware':723,
		'scrapy_splash.SplashMiddleware':725,
		'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddle':810,
		}
	SPIDER_MIDDLEWARES = {
		'scrapy_splash.SplashDeduplicateArgsMiddleware':100,
		}
	DUPEFILTER_CLASS = 'scrapy_splash.SplashAwareDupeFilter'
	HTTPCACHE_STORAGE = 'scrapy_splash.SplashAwareFSCacheStorage'

#在spider项目中添加start_requests方法
from scrapy_splash import SplashRequest
def start_requests(self):
	for url in self.start_urls:
		yidle SplashRequest(url,self.parse)
```
**scrapy 使用 selenium 爬取网页**

**scrapy 分布式爬虫**

大神笔记
http://www.cnblogs.com/zhaof/tag/%E7%88%AC%E8%99%AB/default.html?page=2

