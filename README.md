# comfortable_life
## 1 项目说明
本人想做一个记录个人基本信息的系统，能够快速查询到一些个人的资料。目前准备使用python + Django搭建一个平台，使用mysql存储数据。
## 2 搭建python + Django的环境
python安装参考 [python安装说明](https://github.com/yunfei00/document/blob/master/software_instructions/python.md)
Django环境配置参考 [Django 环境配置](https://github.com/yunfei00/document/blob/master/software_instructions/python_lib/django.md)

# 2. 创建项目
```
django-admin startproject mysite

mysite    --项目容器，名字是自己定义的
|-- manage.py --一个命令行实用程序，允许您以各种方式与此Django项目进行交互
`-- mysite --内部mysite/目录是项目的实际Python包
    |-- wsgi.py      --与WSGI兼容的Web服务器的入口点，用于为您的项目提供服务。
    |-- urls.py      --这个Django项目的URL声明; 您的Django支持的站点的“目录”
    |-- settings.py  --此Django项目的设置/配置
    |-- __init__.py  --空文件，告诉python这个文件夹是python包
    `-- asgi.py

```    
# 3. 启动服务器
```
python3 manage.py runserver
```
默认情况下，该runserver命令在端口8000的内部IP上启动开发服务器。
如果要更改服务器的IP，请将其与端口一起传递。例如，要侦听所有可用的公共IP（如果您正在运行Vagrant或想要在网络上的其他计算机上展示您的工作，这很有用），请使用：
```
python3 manage.py runserver 0:8000
```
0是0.0.0.0的快捷方式

# 4. 创建民意调查应用

```
python3 manage.py startapp polls

polls/
|-- admin.py
|-- apps.py
|-- __init__.py
|-- migrations
|   `-- __init__.py
|-- models.py
|-- tests.py
`-- views.py
```
# 5. 编写第一个视图
1. 视图内容
```
polls/views.py

from django.http import HttpResponse
def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```
2. 视图映射到当前app的url
写完视图，需要显示到网页上，还需要映射的url。
需要在当前app种创建urls映射文件urls.py

```
polls/urls.py

from django.urls import path
from . import views
urlpatterns = [
    path('', views.index, name='index'),
]
```

3. 添加当前app中的url到根目录的url中
```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

4. 遇到问题
Invalid HTTP_HOST header: '192.168.0.100:8000'. You may need to add '192.168.0.100' to ALLOWED_HOSTS.
解决方案：
在我们创建的项目里修改setting.py文件
ALLOWED_HOSTS = ['*']  ＃在这里请求的host添加了*

# 6. 数据库配置
1. 打开  `mysite/settings.py`  ,这是个包含了 Django 项目设置的 Python 模块。

通常，这个配置文件使用 SQLite 作为默认数据库。如果你不熟悉数据库，或者只是想尝试下 Django，这是最简单的选择。Python 内置 SQLite，所以你无需安装额外东西来使用它。
如果你想使用其他数据库，你需要先安装数据库，然后改变设置文件中  `DATABASES`， `default`  项目中的一些键值：

-   **`ENGINE`**  可选值有 
	-  django.db.backends.sqlite3
	- django.db.backends.postgresql
	- django.db.backends.mysql
	- django.db.backends.oracle
-  **`NAME`** 数据库的名称
	如果使用的是 SQLite，数据库将是你电脑上的一个文件，在这种情况下，  [`NAME`](https://docs.djangoproject.com/zh-hans/2.2/ref/settings/#std:setting-NAME)  应该是此文件的绝对路径，包括文件名。默认值  `os.path.join(BASE_DIR,  'db.sqlite3')`  将会把数据库文件储存在项目的根目录。

	mysql 配置，修改原来的DATABASES为以下内容：
	```
	DATABASES = {
	   'default': {
	       'ENGINE': 'django.db.backends.mysql',
	       'NAME': 'mydatabase',
	       'USER': 'mydatabaseuser',
	       'PASSWORD': 'mypassword',
	       'HOST': '127.0.0.1',
	       'PORT': '3306',
	   }
	}
	```

## mysql 遇到问题
django.core.exceptions.ImproperlyConfigured: Error loading MySQLdb module.
Did you install mysqlclient?

安装pip install pymysql   
mysite:
__init__.py 文件加入：
import pymysql
pymysql.install_as_MySQLdb()


~~vim /Users/j00226207/install_program_path/anaconda3/lib/python3.7/site-packages/django/db/backends/mysql/base.py
修改：
if version < (1, 3, 13):
   pass
   '''
   raise ImproperlyConfigured(
       'mysqlclient 1.3.13 or newer is required; you have %s.'
       % Database.__version__
   )
   '''
vim /Users/j00226207/install_program_path/anaconda3/lib/python3.7/site-packages/django/db/backends/mysql/operations.py
修改：
query = query.decode(errors='replace')为：
query = query.encode(errors='replace')~~

其中一些应用程序至少使用了一个数据库表，因此我们需要在使用它们之前在数据库中创建表。为此，请运行以下命令：
```
python manage.py migrate
```
该migrate命令查看INSTALLED_APPS设置并根据mysite/settings.py文件中的数据库设置和应用程序附带的数据库迁移创建任何必要的数据库表（稍后我们将介绍这些表）。

# 7 创建模型（数据表）
```
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```
运行如下命令进行创建：
```
python manage.py makemigrations polls
```

# 8 管理员
(base) yunfei:mysite j00226207$ python manage.py createsuperuser

You have 1 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): polls.
Run 'python manage.py migrate' to apply them.

Username (leave blank to use 'j00226207'): admin
Email address: goodman_yunfei@163.com
Password: 
Password (again): 
Superuser created successfully.




<!--stackedit_data:
eyJoaXN0b3J5IjpbLTcyMzU0NjQ0MCwtMjA1MTUwMTY2LC0xNz
cxNDczMTM4LDIwNTAxMDg1MTMsLTE5NjgzNTA3MzYsNzMxMjk5
NjMwLDk0MzM2MjczMiwtNzI1MzQyNjM3LDEwMDczOTQ1NDEsMT
E5ODM2MjQxLDgzNjg5NzM3MSwtMTg0MzQ2NTIzMiwxNDkwOTkx
OTk4LDExMDE1MDk1MjRdfQ==
-->