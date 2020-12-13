[TOC]



# Django

## django安装

### django安装前置条件

首先确保安装了最新的`python3.x`.  安装`python3.x`一般会附送`pip`.  如果没有，百度搜索怎么安装`pip`.

因为`pip`工具是老外的，所以`pip`的**源**(源可以认为是一个软件仓库)也是国外的。由于GFW的存在，会导致我们在使用pip时候慢的要死。所以我们两步走： 1. 升级pip，2. 更改pip源为清华大学的源。

1. 升级pip:

   `pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U`

2. 更改pip源为清华大学的源:

   `pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple`

### django安装正戏

`pip install django`



## 创建django项目的两种方式

### 图形化界面创建(利用Pycharm专业版)

![](https://raw.githubusercontent.com/sdutvth/django_note/main/images/1.png)

![](https://raw.githubusercontent.com/sdutvth/django_note/main/images/2.png)

但pycharm部分版本创建出来的django是有问题的。

django2.x版本在`projectname/projectname/settings.py` 文件开头，应该写上`import os` 用来导包。不然下面用到了`os`模块上面却没导入会出问题。

但django3修复了这个问题。无需导入os。

### 命令创建

`django-admin startproject projectname`

注意，projectname是自己起的名字。不要跟python, django等重名，要跟业务相关。

### django项目的目录结构

`test_django  # 外层目录
├── manage.py # 对django项目的很多操作都要通过manage.py
└── test_django # 存放主项目的配置、路由等
    ├── __init__.py
    ├── asgi.py # 服务器
    ├── settings.py # 配置文件
    ├── urls.py # 路由系统
    └── wsgi.py # 服务器`

## django项目的启动

### 命令启动

进入到项目与`manage.py`同级的目录。

执行 `python manage.py runserver 127.0.0.1:22222`

这代表我们启动了django项目，并且监听本地的22222端口。在浏览器访问的时候，只需要输入：

`127.0.0.1:22222`就能访问到我们创建的项目了。

### pycharm图形化界面启动

pycharm代码区上面有个绿色标志，点就完事了。

![](https://raw.githubusercontent.com/sdutvth/django_note/main/images/3.png)



## Django学习的重点

- django常用操作
- 路由 - 负责把url交给视图函数处理
- 视图 - 负责业务逻辑
- 模板 - 负责渲染页面
- ORM - 负责与数据库打交道

## Django常用操作

### django的app

#### app概念

app是个啥？可以认为是django项目中的某个独立模块，对模块/业务线进行封装。这样有利于我们分头开发，而且出了问题可以快速定位，好处多多。

#### app创建

`python manage.py startapp appname`

#### app的目录结构

`test_django
├── manage.py
├── test_app # 这里才是刚创建的app
│   ├── __init__.py
│   ├── admin.py # 配置django的后台管理，操作数据库
│   ├── apps.py 
│   ├── migrations # 数据库相关
│   │   └── __init__.py
│   ├── models.py # 与数据库相关的库
│   ├── tests.py # 单元测试
│   └── views.py # 写业务代码
└── test_django # 忽略，刚讲过了
    ├── __init__.py
    ├── __pycache__
    │   ├── __init__.cpython-37.pyc
    │   └── settings.cpython-37.pyc
    ├── asgi.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py`

#### app冷知识

app创建好之后，这个目录结构和命名，只有`views.py`可以变，其余都不能变。



## Django路由系统

### 常规路由写法

打开`test_jango/test_django/urls.py`, 改成如下代码：

```python
from django.contrib import admin
from django.urls import path
# 导入HttpResponse, 用来返回合乎要求的HttpResponse
from django.shortcuts import HttpResponse

urlpatterns = [
    path('admin/', admin.site.urls),
    # 下面test是新建的路由
    # path的第一个参数是url, 第二个参数是能返回合乎http协议的内容的函数
    # 这里我省事儿就直接写了个匿名函数
    path('test', lambda request: HttpResponse('in test page.'))
]
```

我们是这么访问这个路由的：



### 动态路由

### 路由分发

### 别名反向生成URL