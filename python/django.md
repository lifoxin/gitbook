## 继承 AbstractUser 拓展用户模型

models.py
```python
from django.db import models
from django.contrib.auth.models import AbstractUser
from django.core.validators import MinLengthValidator

class User(AbstractUser):                                                                     │
    mobile = models.CharField(max_length=11, validators=[MinLengthValidator(11)])
```

settings.py
```
AUTH_USER_MODEL = 'users.User'  # 应用名称.模型类名称
```
views.py
```python
def register_hander(request):
    user = User.objects.all()
    if request.method == 'POST':
        form = LoginForm(request.POST)
        if form.is_valid():
            cleaned_data = form.cleaned_data
            username = cleaned_data['username']
            email = cleaned_data['email']
            mobile = cleaned_data['mobile']
            password = cleaned_data['password']
            user = User.objects.create_superuser(
                    username=username, email=email,
                    mobile=mobile, password=password
                    )
            user.is_staff = True # 可选，有权限登录admin
            user.save()
            print("注册成功")
            return render(request,'user/tips.html')
        else:
            print("注册不成功")
            return render(request,'user/error.html')
    return redirect('/')
    
def register(request):
    return render(request,'user/register.html')
```

创建用户实例

    user = User.objects.create_uer()
    user.is_staff = True         #此行可选，让用户有权登录admin
    user.save()

用户登录无需手动与数据库数据对比，无需对密码执行加密对比操作

    from django.contrib.auth import authenticate
    user = authenticate(username=username, password=password)

实现能让用使用注册用户名、手机号或者邮箱完成登录验证：
在自定义工具目录util中继承ModelBackend类，重写authenticate方法
```python
from django.contrib.auth.backends import ModelBackend
import re
from users.models import User

class MeiduoModelBackend(ModelBackend):
    def authenticate(self, request, username=None, password=None, **kwargs):
        try:
            # get方法查询到数据返回用户对象，未查询到则报错
            user = User.objects.get(username=username)
        except:
            try:
                user = User.objects.get(mobile=username)
            except:
                return None
        # 判断密码
        if user.check_password(password):
            return user
        else:
            return None
```

保持用户登录状态无需手动创建session，把用户ID存入session中。

    from django.contrib.auth import login
    login(request, user)

退出登录操作自动删除session

    from django.contrib.auth import logout
    logout(request)

## 部署上线 nginx + uwsgi +django

[详解](http://django.jeffhardy.cn/part6/6.html)

pip install uwsig

uwsig.ini

	[uwsgi]
	socket=127.0.0.1:8000
	#http=127.0.0.1:8001
	chdir=/home/felix/typeidea/
	wsgi-file=typeidea/wsgi.py
	processes=4
	threads=2
	master=True
	pidfile=/tmp/uwsgi.pid
	daemonize=/tmp/uwsgi.log

nginx.conf

	server {
			listen      80;
			server_name  blog.jeffhardy.cn;
			location / {
				 include uwsgi_params;
				 uwsgi_pass 127.0.0.1:8000;
				 uwsgi_param UWSGI_SCRIPT typeidea/wsgi.py;
				 uwsgi_param UWSGI_CHDIR /home/felix/typeidea;
				 client_max_body_size 35m;
			}
			location /static {
			   alias /home/felix/typeidea/themes/default/static/;
			}
	}

运行

uwsig --ini uwsig.ini && /usr/local/nginx/sbin/nginx 

**安装django版本**

    pip install django==1.11

**创建项目**

    django-admin startproject HelloWorld  

**创建应用**

    cd HelloWorld
    python manage.py startapp django_web

**创建超级用户**
    
    python manage.py createsuperuser
    pwd：

**创建迁移文件**
    
    python manage.py makemigrations 

**迁移数据表**
    
    python manage.py migrate

**运行项目**
    
    python manage.py runserver 

**常见命令**

```python
django-admin startproject project-name
python manage.py startapp app-name
python manage.py createsuperuser

python manage.py makemigrations
python manage.py migrate
python manage.py flush
python manage.py dbshell
python manage.py dumpdata appname > appname.json
python manage.py loaddata appname.json

python manage.py shell
python manage.py runserver 0.0.0.0:8000
```
添加应用到项目
>新定义的app加到settings.py中的INSTALL_APPS中

django的学习操作
```
from django.conf.urls import url, include
from django.contrib import admin

urlpatterns = [
url(r'^add/(\d+)/(\d+)/$', 'calc.views.add2', name='add2'),
]

>>>from django.core.urlresolvers import reverse
>>>reverse('add2',args(4,5))
'/add/4/5/'

reverse 接收 url 中的 name 作为第1个参数，我们在代码中就
可以通过 reverse() 来获取对应的网址，只要对应的 url 的name不改，
就不会改代码中的网址。

网页可以调用url

namespace的应用
url(r'^booktest/', include('booktest.urls', namespace='booktest')),
{% url 'booktest.add2' 4 5 %} 

不带参数的：
{% url 'name' %}
带参数的
{% url 'name' 参数 %}
<a href="{% url 'add2' 4 5 %}">link</a>


视图

from django.http import HttpResponse
from django.shortcuts import render

def add2(request):
    a = request.GET['a']
    b = request.GET['b']
    c = int(a)+int(b)
    return HttpResponse(str(c))

HttpResponse，它是用来向网页返回内容的，
只不过 HttpResponse 是把内容显示到
网页上。

from django.shortcuts import render
from models import BookInfo

def index(reqeust):
    booklist = BookInfo.objects.all()
    return render(reqeust, 'booktest/index.html', {'booklist': booklist})


def detail(reqeust, id):
    book = BookInfo.objects.get(pk=id)
    return render(reqeust, 'booktest/detail.html', {'book': book})

render,他是传递请求和模块，渲染页面，返回浏览器

GET方法

dict.get('键',default)
或简写为
dict['键']

?a=1&b=2
request.GET['键']
?a=1&a=2&b=3
requests.GET.getlist['键']

POST方法

postTest1.html
<form method="post" action="/postTest2/">
    姓名：<input type="text" name="uname"/><br>
    密码：<input type="password" name="upwd"/><br>
    性别：<input type="radio" name="ugender" value="1"/>男
    <input type="radio" name="ugender" value="0"/>女<br>
    爱好：<input type="checkbox" name="uhobby" value="胸口碎大石"/>胸口碎大石
    <input type="checkbox" name="uhobby" value="跳楼"/>跳楼
    <input type="checkbox" name="uhobby" value="喝酒"/>喝酒
    <input type="checkbox" name="uhobby" value="爬山"/>爬山<br>
    <input type="submit" value="提交"/>
</form>

def postTest2(request):
    uname=request.POST['uname']
    upwd=request.POST['upwd']
    ugender=request.POST['ugender']
    uhobby=request.POST.getlist('uhobby')
    context={'uname':uname,'upwd':upwd,'ugender':ugender,'uhobby':uhobby}
    return render(request,'booktest/postTest2.html',context)

session方法

request.session['键']
request.session.flush()
request.session.get('键')
del request.session['键']
request.session.clear()

重定向到 /, 即输入 xxx:8000/red1，结果为 xxx:8000/
from django.shortcuts import redirect
from django.core.urlresolvers import reverse

def red1(request):
    return redirect('/')

def index(request):
    return redirect(reverse('booktest:index2'))

子类JsonResponse，返回json数据，一般用于异步请求
from django.http import JsonResponse
def index2(requeset):
    return JsonResponse({'list': 'abc'})

显示400、500、404错误页面，在templates创建404.html
DEBUG = False
ALLOWED_HOSTS = ['*', ]

404.html
<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>
找不到了
<hr/>
{{request_path}}
</body>
</html>

from django.shortcuts import *

得到对象或返回404
def detail(request, id):
    try:
        book = get_object_or_404(BookInfo, pk=id)
    except BookInfo.MultipleObjectsReturned:
        book = None
    return render(request, 'booktest/detail.html', {'book': book})

得到列表或返回404
def detail(request, id):
    try:
        book = get_object_or_404(BookInfo, pk=id)
    except BookInfo.MultipleObjectsReturned:
        book = None
    return render(request, 'booktest/detail.html', {'book': book})

状态保持

http协议是无状态的：每次请求都是一次新的请求，不会记得之前通信的状态
客户端与服务器端的一次通信，就是一次会话
实现状态保持的方式：在客户端或服务器端存储与会话有关的数据
存储方式包括cookie、session，会话一般指session对象
使用cookie，所有数据存储在客户端，注意不要存储敏感信息
推荐使用sesison方式，所有数据存储在服务器端，在客户端cookie中存储session_id
状态保持的目的是在一段时间内跟踪请求者的状态，可以实现跨页面访问当前请求者的数据
注意：不同的请求者之间不会共享这个数据，与请求者一一对应

启用session

项INSTALLED_APPS列表中添加：
'django.contrib.sessions',

项MIDDLEWARE_CLASSES列表中添加：
'django.contrib.sessions.middleware.SessionMiddleware',

存储session，将缓存和数据库同时使用：优先从本地缓存中获取，如果没有则从数据库中获取
SESSION_ENGINE='django.contrib.sessions.backends.cached_db'

使用Redis缓存session

安装包
pip install django-redis-sessions

修改settings中的配置，增加如下项
SESSION_ENGINE = 'redis_sessions.session'
SESSION_REDIS_HOST = 'localhost'
SESSION_REDIS_PORT = 6379
SESSION_REDIS_DB = 0
SESSION_REDIS_PASSWORD = ''
SESSION_REDIS_PREFIX = 'session'

管理redis的命令
启动：sudo redis-server /etc/redis/redis.conf
停止：sudo redis-server stop
重启：sudo redis-server restart
redis-cli：使用客户端连接服务器
keys *：查看所有的键
get name：获取指定键的值
del name：删除指定名称的键

在views.py文件中创建视图
from django.shortcuts import render, redirect
from django.core.urlresolvers import reverse

def index(request):
    uname = request.session.get('uname')
    return render(request, 'booktest/index.html', {'uname': uname})

def login(request):
    return render(request, 'booktest/login.html')

def login_handle(request):
    request.session['uname'] = request.POST['uname']
    return redirect(reverse('main:index'))

def logout(request):
    # request.session['uname'] = None
    # del request.session['uname']
    # request.session.clear()
    request.session.flush()
    return redirect(reverse('main:index'))

配置url
主url：
from django.conf.urls import include, url
urlpatterns = [
    url(r'^', include('booktest.urls', namespace='main'))
]

应用url：
from django.conf.urls import url
from . import views
urlpatterns = [
    url(r'^$', views.index, name='index'),
    url(r'login/$', views.login, name='login'),
    url(r'login_handle/$', views.login_handle, name='login_handle'),
    url(r'logout/$', views.logout, name='logout')
]

创建模板index.html
<!DOCTYPE html>
<html>
<head>
    <title>首页</title>
</head>
<body>
你好：{{uname}}
<hr/>
<a href="{%url 'main:login'%}">登录</a>
<hr/>
<a href="{%url 'main:logout'%}">退出</a>
</body>
</html>

创建模板login.html
<!DOCTYPE html>
<html>
<head>
    <title>登录</title>
</head>
<body>
<form method="post" action="/login_handle/">
    <input type="text" name="uname"/>
    <input type="submit" value="登录"/>
</form>
</body>
</html>

模板

默认配置下，Django 的模板系统会自动找到app下面的
templates文件夹中的模板文件。

修改settings.py文件，设置TEMPLATES的DIRS值
'DIRS': [os.path.join(BASE_DIR, 'templates')],

在模板中访问视图传递的数据
{{ 输出值，可以是变量，也可以是对象.属性 }}
{% 执行代码段 %}

(定义模板)[http://django.jeffhardy.cn/part4/2.html]

防csrf的使用

修改settings.py文件
启用'django.middleware.csrf.CsrfViewMiddleware'中间件

<form>
{% csrf_token %}
...
</form>

取消csrf保护

from django.views.decorators.csrf import csrf_exempt
@csrf_exempt
def csrf2(request):
    ...

验证码

...

静态文件

项目中的CSS、图片、js都是静态文件

STATIC_URL = '/static/'
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),
]

mysite/static/myapp/

模板中调用
/static/myapp/myexample.jpg

{ % load static from staticfiles %}
<img src="{ % static "myapp/myexample.jpg" %}" alt="My image"/>

上传图片

pic=models.ImageField(upload_to='cars/')
注意：如果属性类型为ImageField需要安装包Pilow
pip install Pillow==3.4.1

图片上传后，会被保存到“/static/media/cars/图片文件”

打开settings.py文件，增加media_root项
MEDIA_ROOT=os.path.join(BASE_DIR,"static/media")


<html>
<head>
    <title>文件上传</title>
</head>
<body>
    <form method="post" action="upload/" enctype="multipart/form-data">
        <input type="text" name="title"><br>
        <input type="file" name="pic"/><br>
        <input type="submit" value="上传">
    </form>
</body>
</html>


from django.conf import settings

def upload(request):
    if request.method == "POST":
        f1 = request.FILES['pic']
        fname = '%s/cars/%s' % (settings.MEDIA_ROOT,f1.name)
        with open(fname, 'w') as pic:
            for c in f1.chunks():
                pic.write(c)
        return HttpResponse("ok")
    else:
        return HttpResponse("error")

模型

pip install mysql-python
create databases test2 charset=utf8

settings.py文件，修改DATABASES项
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'test2',
        'USER': '用户名',
        'PASSWORD': '密码',
        'HOST': '数据库服务器ip，本地可以使用localhost',
        'PORT': '端口，默认为3306',
    }
}

模型查询有点多
filter(键1=值1,键2=值2)
filter(title__contains="%")=>where title like '%\%%'，表示查找标题中包含%的

查询集的缓存
querylist=Entry.objects.all()
print([e.title for e in querylist])

使用aggregate()函数返回聚合函数的值
函数：Avg，Count，Max，Min，Sum
from django.db.models import Max
maxDate = list.aggregate(Max('bpub_date'))
count的一般用法：
count = list.count()
```
参考
http://django.jeffhardy.cn

复杂的模块要看文档
导入包，方法的使用


