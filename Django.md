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

在`projectname/projectname/settings.py` 文件开头，应该写上`import os` 用来导包。不然下面用到了`os`模块上面却没导入会出问题。

### 命令创建

`django-admin startproject projectname`

注意，projectname是自己起的名字。不要跟python, django等重名，要跟业务相关。



## django项目的启动

### 命令启动

进入到项目与`manage.py`同级的目录。

执行 `python manage.py runserver 127.0.0.1:22222`

这代表我们启动了django项目，并且监听本地的22222端口。在浏览器访问的时候，只需要输入：

`127.0.0.1:22222`就能访问到我们创建的项目了。

