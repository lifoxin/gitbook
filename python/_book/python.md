### 定时发送邮件
25 20 * * 1,3,5 bash /home/felix/crawl.sh >> /dev/null 2>&1
```shell
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

### mail发送邮件
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
### os模块
* os.getcwd()  #返回当前的工作目录
* os.chdir('/server/accesslogs')   # 修改当前的工作目录
* os.system('mkdir today')   # 执行系统命令 mkdir 
### 爬取图片
```python
import requests
r = requests.get("https://github.com/favicon.ico")
with open('favicon.ico', 'wb') as f:
    f.write(r.content)
```
### 知乎的网站需要添加 headers
```python
import requests
headers = {
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36'
}
r = requests.get("https://www.zhihu.com/explore", headers=headers)
print(r.text)
```
### get请求，提交数据
```python
import requests
data = {
    'name': 'germey',
    'age': 22
}
r = requests.get("http://httpbin.org/get", params=data)
print(r.text)
 ```
### post请求，提交数据
```python
import requests
data = {'name': 'germey', 'age': '22'}
r = requests.post("http://httpbin.org/post", data=data)
print(r.text)
```
### post请求，提交文件
```
import requests
files = {'file': open('favicon.ico', 'rb')}
r = requests.post('http://httpbin.org/post', files=files)
print(r.text)
```
- exit() if not r.status_code == requests.codes.ok else print('Request Successfully')
- 如果我们想判断结果是不是 404 状态，可以用 requests.codes.not_found 来比对。

### 正则表达式
* re.S 用于.匹配换行 ，？是非贪婪匹配
* match是从头开始匹配，
* search是匹配表达式第一个内容
```python
    result = re.search('<li.*?active.*?singer="(.*?)">(.*?)</a>', html, re.S)
```
* group是匹配出第n个感兴趣的内容。
```python
    if result:
        print(result.group(1), result.group(2))  
```
* findall全文匹配
```
    results = re.findall('<li.*?href="(.*?)".*?singer="(.*?)">(.*?)</a>', html, re.S)
    results = re.findall('<li.*?>(.*?)</li>', html, re.S)
    for result in results:
        print(result.strip())
```
* sub是替换用法，把a节点换掉，再截取。
```python
    content = '54aK54yr5oiR54ix5L2g'
    content = re.sub('\d+', '', content)
    html = re.sub('<a.*?>|</a>', '', html)
```
* compile() 方法，这个方法可以讲正则字符串编译成正则表达式对象，以便于在后面的匹配中复用。
```python
    content1 = '2016-12-15 12:00'
    pattern = re.compile('\d{2}:\d{2}')
    result1 = re.sub(pattern, '', content1)
```
### [xpath](http://www.w3school.com.cn/xpath/index.asp) 的用法
//input[contains(@name,'na')]         查找name属性中包含na关键字的页面元素
### [json](http://www.w3school.com.cn/json/index.asp)
* json.dumps(): 对数据进行编码。
* json.loads(): 对数据进行解码。
* 写入 JSON 数据
 ```python
    with open('data.json', 'w') as f:
        json.dump(data, f)
```
* 读取数据
```python
    with open('data.json', 'r') as f:
        data = json.load(f)
```
* 保存 json
```python
    ofile.write(json.dumps(record) + '\n')
    ofile.flush()
    ofile.close()
```
### zip，实现把两个list组合成一个dict
- 格式为： dict(zip(keys,vals))
```python
    list1=[1,45,232,45,666,64]
    list2=["ss","kein","tom","sda","qq","da"]
    d=dict(zip(list1,list2))
    print(d)
    {1: 'ss', 45: 'sda', 232: 'tom', 666: 'qq', 64: 'da'}
```
### map函数的原型是map(function, iterable, …)，它的返回结果是一个列表。
- 参数function传的是一个函数名，可以是python内置的，也可以是自定义的。
- 参数iterable传的是一个可以迭代的对象，例如列表，元组，字符串这样的。

* str.isdigit() 方法检测字符串是否只由数字组成。
    - 如果字符串只包含数字则返回 True 否则返回 False。

### selenium对web各元素的操作首先就要先定位元素，定位元素的方法主要有以下几种：
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

