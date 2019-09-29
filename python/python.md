**python垃圾回收机制**

引用计数机制为主，标记-清除和分代收集两种机制为辅

当引用计数为0时，该对象生命就结束了。

优点：

1. 简单
2. 实时性：一旦没有引用，内存就直接释放了。不用像其他机制等到特定时机。实时性还带来一个好处：处理回收内存的时间分摊到了平时。

缺点：

1. 维护引用计数消耗资源
2. 循环引用

**装饰器**

修改其他函数的功能的函数 [详解](https://www.runoob.com/w3cnote/python-func-decorators.html)

```python
from functools import wraps
 
def a_new_decorator(a_func):
    @wraps(a_func)
    def wrapTheFunction():
        print("I am doing some boring work before executing a_func()")
        a_func()
        print("I am doing some boring work after executing a_func()")
    return wrapTheFunction
 
@a_new_decorator
def a_function_requiring_decoration():
    """Hey yo! Decorate me!"""
    print("I am the function which needs some decoration to "
          "remove my foul smell")
 
print(a_function_requiring_decoration.__name__)
# Output: a_function_requiring_decoration
```

**类的方法**

```python
class A(object):  
    def foo(self,x):  
    #类实例方法  
        print("executing foo(%s,%s)"%(self,x))  
  
    @classmethod  
    def class_foo(cls,x):  
    #类方法  
        print("executing class_foo(%s,%s)"%(cls,x)
  
    @staticmethod  
    def static_foo(x):  
    #静态方法  
        print("executing static_foo(%s)"%x)

#调用
a = A()  
a.foo(1)          //executing foo(<__main__.A object at 0xb77d67ec>,1)

a.class_foo(1)    //executing class_foo(<class '__main__.A'>,1)  
A.class_foo(1)    //executing class_foo(<class '__main__.A'>,1)  
   
a.static_foo(1)   //executing static_foo(1)  
A.static_foo(1)   //executing static_foo(1)
```

**安装virtualenv**

pip install virtualenv

创建test项目，使用虚拟环

1. virtualenv --no-site-packages venv  #--no-site-packages表示不适用python3环境的模块
1. source venv/bin/activate

退出虚拟环境

deactivate

在venv环境下，用pip安装的包都被安装到venv这个环境下，系统Python环境不受任何影响
 
**定时任务**

25 20 * * 1,3,5 bash /home/felix/crawl.sh >> /dev/null 2>&1
```bash
#!/bin/bash
#script of /home/felix/crawl.sh
echo "crawling ..."
python3 /home/felix/test/spider/qianchenthread.py > /tmp/yulu.text 2>&1
echo "crawl is over,then send email to others ..."
python3 /home/felix/test/rookie/sendMail.py
if [ $? -ne 0 ]
then
    echo "$(date +"%F %H:%M:%S")" >> sendTime
    echo "send email success !"
fi
```
**mail**
```python
import smtplib
from email.mime.text import MIMEText

host = 'smtp.163.com'
port = 465
sender = '18344589481@163.com'
pwd = '授权密码'

def sentemail():
    receivers = ["805986238@qq.com","381407211@qq.com"]
    with open("/tmp/yulu.text") as f:
        content = f.read()
#   msg = MIMEText(body, 'html') 根据邮件内容选择格式,这里选择 文本格式
    msg = MIMEText(content,_subtype='plain',_charset='utf-8')
    msg['subject'] = '前程无忧运维工作'
    msg['from'] = sender
    for receiver in receivers:
        msg['to'] = receiver
        s = smtplib.SMTP_SSL(host, port)
        # 注意！如果是使用SSL端口，这里就要改为SMTP_SSL
        s.login(sender, pwd)
        s.sendmail(sender, receiver, msg.as_string().encode())
    print ('Done.sent email success')

if __name__ == '__main__':
    sentemail()

```
**爬虫**
```python
import requests
r = requests.get("https://github.com/favicon.ico")
with open('favicon.ico', 'wb') as f:
    f.write(r.content)

import requests
headers = {
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36'
}
r = requests.get("https://www.zhihu.com/explore", headers=headers)
print(r.text)
```
**get**
```python
import requests
data = {
    'name': 'germey',
    'age': 22
}
r = requests.get("http://httpbin.org/get", params=data)
print(r.text)
 ```
**post**
```python
import requests
data = {'name': 'germey', 'age': '22'}
r = requests.post("http://httpbin.org/post", data=data)
print(r.text)

import requests
files = {'file': open('favicon.ico', 'rb')}
r = requests.post('http://httpbin.org/post', files=files)
print(r.text)
```
- exit() if not r.status_code == requests.codes.ok else print('Request Successfully')
- 如果我们想判断结果是不是 404 状态，可以用 requests.codes.not_found 来比对。

**re**
1. re.S 用于.匹配换行 ，？是非贪婪匹配
1. match是从头开始匹配，
1. search是匹配表达式第一个内容
1. findall全文匹配
1. sub替换
1. compile匹配规则

```python
result = re.search('<li.*?active.*?singer="(.*?)">(.*?)</a>', html, re.S)
if result:
	print(result.group(1), result.group(2))  
results = re.findall('<li.*?href="(.*?)".*?singer="(.*?)">(.*?)</a>', html, re.S)
results = re.findall('<li.*?>(.*?)</li>', html, re.S)
for result in results:
	print(result.strip())
content = '54aK54yr5oiR54ix5L2g'
content = re.sub('\d+', '', content)
html = re.sub('<a.*?>|</a>', '', html)
content1 = '2016-12-15 12:00'
pattern = re.compile('\d{2}:\d{2}')
result1 = re.sub(pattern, '', content1)
```
**xpath**
```
//input[contains(@name,'na')]     
查找name属性中包含na关键字的页面元素
```
**json**

1. json.dumps(): 对数据进行编码。
1. json.loads(): 对数据进行解码。

```python
with open('data.json', 'w') as f:
	json.dump(data, f)

with open('data.json', 'r') as f:
	data = json.load(f)

ofile.write(json.dumps(record) + '\n')
ofile.flush()
ofile.close()
```
**zip**

1. 实现把两个list组合成一个dict
1. 格式为： dict(zip(keys,vals))

```python
list1=[1,45,232,45,666,64]
list2=["ss","kein","tom","sda","qq","da"]
d=dict(zip(list1,list2))
print(d)
{1: 'ss', 45: 'sda', 232: 'tom', 666: 'qq', 64: 'da'}
```
**map**

1. 函数的原型是map(function, iterable, …)，它的返回结果是一个列表。
1. 参数function传的是一个函数名，可以是python内置的，也可以是自定义的。
1. 参数iterable传的是一个可以迭代的对象，例如列表，元组，字符串这样的。

**all**
```
all(iterable)
all() 函数用于判断给定的可迭代参数 iterable 中的所有元素是否都为 TRUE，如果是返回 True，否则返回 False。
```
**selenium**

- 对web各元素的操作首先就要先定位元素，定位元素的方法主要有以下几种：

```
通过id定位元素：find_element_by_id("id_vaule")
通过name定位元素：find_element_by_name("name_vaule")
通过tag_name定位元素：find_element_by_tag_name("tag_name_vaule")
通过class_name定位元素：find_element_by_class_name("class_name")
通过css定位元素：find_element_by_css_selector();用css定位是比较灵活的
通过xpath定位元素：find_element_by_xpath("xpath")
通过link定位：find_element_by_link_text("text_vaule")或者find_element_by_partial_link_text()
   以百度首页为例：下面是百度输入框的html代码，可以通过firebug或者谷歌的审查元素或得

<input type="text"name="wd" id="kw1" maxlength="100"style="width:474px;"
 autocomplete="off">
1.通过id定位，则百度的输入框即可表示为：find_element_by_id("kw1")
2.通过name定位则可以表示为：find_element_by_name("wd")
3.通过tag_name定位：input其实就是tag_name（标签名），同样也可以表示成：
find_element_by_tag_name("input")
下面是“百度一下”按钮的html
<span  class="btn_wr">
<input type="submit" value="百度一下" id="su1" class="btn" onmousedown=
"this.className='btnbtn_h'" onmouseout="this.className='btn'">
</span>
4.通过class_name定位，“百度一下”按钮则可以表示成find_element_by_class_name("btn_wr")
5.通过css定位，这个比较灵活，想要完全弄懂，花费的时间是
比较多的，个人觉得没有必要

百度输入框

<input
 type="text" name="wd"id="kw1" maxlength="100"style="width:474px;" autocomplete="off">
如取id，百度输入框则可以表示为：find_element_by_css_selector("a[id=\"kw1\"]")
如取name，又可以表示为：find_element_by_css_selector("a[name=\"wd\"]") 
<aonclick="queryTab(this);" mon="col=502&pn=0" title="web" href="http://www.baidu.com/">网页</a>
 还可以用title，如百度的网页链接可以表示为find_element_by_css_selector("a[title=\"web\"]")
<a class="RecycleBinxz" href="javascript:void(0);">

还也同样可以用class，上面的代码有可以用find_element_by_css_selector("a.RecycleBin")
6.通过XPath定位

首先我们要了解XPath是上面东西，XPath是一种在XML

xpath:attributer（属性）
driver.find_element_by_xpath("//input[@id='kw1']")
表示input标签下id =kw1的元素

xpath:idRelative（id相关性）
driver.find_element_by_xpath("//div[@id='fm']/form/span/input")
表示在/form/span/input层级标签下有个div标签的id=fm的元素

driver.find_element_by_xpath("//tr[@id='check']/td[2]")
表示id为'check'的tr，定闪他里面的第2个td

xpath:position（位置）
driver.find_element_by_xpath("//input")
driver.find_element_by_xpath("//tr[7]/td[2]")
表示第7个tr里面的第2个td

xpath: href（水平参考）
driver.find_element_by_xpath("//a[contains(text(),'网页')]")
表示在a标签下有个文本（text）包含（contains）'网页' 的元素

xpath:link
dri
表示有个叫a的标签，他有个链接href='http://www.baidu.com/的元素

7.通过link定位
有时候不是一个输入框也不是一个按钮，而是一个文字链接，我们可以通过link

#coding=utf-8
from seleniumimport webdriver
import time
df = webdriver.Firefox()   #选择firefox浏览器
df.get("http://www.baidu.com")   #打开百度网页
time.sleep(2)        #暂停2秒，不是毫秒
df.find_element_by_link_text("贴 吧").click()     #点击贴吧链接
time.sleep(2)
df.quit()           #关闭浏览器
Partial Link Text 定位
通过部分链接定位，这个有时候也会用到，我还没有想到很好的用处。拿上面的例子，我可以只用链接的一部分文字进行匹配：
browser.find_element_by_partial_link_text("贴").click()

通过find_element_by_partial_link_text()函数，我只用了“贴”字，脚本一样找到了"贴吧"的链接
```

