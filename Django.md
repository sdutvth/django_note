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

### 路由原理

路由其实就是类似于多个if-else if-else块串联的情况。一旦匹配到之后，就不往下匹配了。匹配的方式可以用正则，也可以用全文匹配，django对于这两种需求都提供了对应的方法。

根据路由的串联原则，我们可以设置一个404页面，一旦匹配失败就可以跳转到404页面。

```python
from django.contrib import admin
from django.urls import path, re_path, include
from django.shortcuts import Http404, HttpResponse


urlpatterns = [
    path('admin/', admin.site.urls),
    path('app01', include('test_app.urls')),
    # 404页面写法
    re_path('', lambda request: HttpResponse('404 NOT FOUND!'))
]
```

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

![](https://raw.githubusercontent.com/sdutvth/django_note/main/images/4.png)

所谓的常规路由，就是写死了的字符串。只要url中的字符串能和我们写在代码里的字符串完全匹配，就能访问到。

### 动态路由

#### 动态路由的概念

所谓动态路由，即对于某一类具有共同特征的url，我们映射到同一个视图函数进行处理。

比如以test为开头的url，同意返回Hello, test. 的示例如下：

url:

`http://127.0.0.1:8000/test12`

`http://127.0.0.1:8000/test22`

针对这两个url，我们都会返回：

`hello, test!`

#### 动态路由的实现

##### 总览

动态路由在django2.x+中的实现方式有2种。一种是在path中通过django2.x的特性来实现。另外一种就是在re_path中用正则表达式的形式来实现。

##### path的实现

re_path太难写了，有没有更方便的？当然有，path就提供了。

而且，从源头上就解决了判定从url中取到的是个什么类型的玩意儿的问题！

新版本的path引入了一个**转换器**的概念。

转换器是个啥？ 相当于是正则表达式的简约版，并且能把匹配到的东西搞成指定的类型。

举个例子：

`<int:age>` 这就是一个转换器。能截取一个int类型的值，赋值给age. 即视图函数的一个名为age的命名参数。

Django默认支持下面五种转换器：

- str,匹配除了路径分隔符（`/`）之外的非空字符串，这是默认的形式，即不写类型则默认str。
- int,匹配正整数，包含0。
- slug,匹配字母、数字以及横杠、下划线组成的字符串。
- uuid,匹配格式化的uuid，如 075194d3-6885-417e-a8a8-6c931e272f00。
- path,匹配任何非空字符串，包含了路径分隔符`/`。

写点demo:

```python
from django.contrib import admin
from django.urls import path, re_path
# 导入HttpResponse, 用来返回合乎要求的HttpResponse
from django.shortcuts import HttpResponse

urlpatterns = [
    path('admin/', admin.site.urls),
    path('test<int:age>/<user>', lambda request, age, user: HttpResponse(age))
]
```

这样的话，我们的url就可以写成：

`http://127.0.0.1:8000/test22/222`

这样的话，就能匹配到了。

在使用过程中，强烈建议两个转换器/两个正则之间加路径分隔符`/`。 这样不容易混淆。

注意写法的一致性，前面写了age, 后面的参数就要叫age。因为这相当于是python中的命名参数传参的形式。

##### re_path的实现

re_path的实现方式跟django1.x的urls的实现方式类似。

re_path跟path的不同之处就在于，re_path把字符串当做一个正则表达式来处理。

当然，正则表达式的动态匹配部分必须被`()`包裹。

一个`()`代表了一个正则块，在正则块外，按照path的规则写，在正则块内，直接写正则即可。

一个正则块对应着视图函数的一个参数。别忘了，视图函数的第一个参数永远是request对象。所以正则块对应的参数顺延即可。

另外有一点需要特别注意， re_path是正则表达式匹配，只要符合所写正则的url都可以通过。所以，对于`xx`来讲，`xxxxx`这个url是合法的。为了避免这种情况，我们可以在re_path正则表达式的开头和结尾分别加上`^`和`$`代表开始和结束，这样就可以做到精准匹配了。

举个例子：

这是我们的test_django/test_django/urls.py的内容：

```python
from django.contrib import admin
from django.urls import path, re_path
# 导入HttpResponse, 用来返回合乎要求的HttpResponse
from django.shortcuts import HttpResponse

urlpatterns = [
    path('admin/', admin.site.urls),
    re_path('test([1-3]{2})', lambda request, re_str: HttpResponse('hello, test!'))
]
```

这代表着我们在输入url时，输入`http://127.0.0.1:8000/test22`和`http://127.0.0.1:8000/test12`都是ok的。因为他们符合我们写的这个正则。

注意到了吗？我们的lambda匿名函数设置了2个参数。因为第一个参数是request, 第二个参数是我们的正则表达式块匹配到的内容，所以要写2个参数。参数re_str对于url: `http://127.0.0.1:8000/test22`来讲，会匹配到22. 这种写法跟python的位置参数差不多。



再举个例子：

同样是test_django/test_django/urls.py中的内容：

```python
from django.contrib import admin
from django.urls import path, re_path
# 导入HttpResponse, 用来返回合乎要求的HttpResponse
from django.shortcuts import HttpResponse

urlpatterns = [
    path('admin/', admin.site.urls),
    re_path('test([1-3]{2})(\w+)', lambda request, re_str, t: HttpResponse('hello, test!'))
]
```

我们输入`http://127.0.0.1:8000/test22e`可以匹配上。 请注意，re_path包含了2个正则块，所以lambda匿名函数需要3个参数。

最后再强调一点，我们在正则块外的规则同path，即，可以用`/`来分隔。

还有一种语法，是给re_path的正则块命个名，当然代价就是在接受的时候参数名要跟命名是相同的。

```python
from django.contrib import admin
from django.urls import path, re_path
# 导入HttpResponse, 用来返回合乎要求的HttpResponse
from django.shortcuts import HttpResponse

urlpatterns = [
    path('admin/', admin.site.urls),
    re_path('test(?P<tes>[1-3]{2})(?P<ss>\w+)', lambda request, ss, tes: HttpResponse('hello, test!'))
]
```

注意看，url还是那个url，只不过lambda表达式中的参数已经是确定的名字了。

**最后说一下，re_path的命名用法和非命名用法不能混用，否则会出问题。**

### 路由分发

所谓的路由分发，其实就是把路由映射关系从整个django项目的`test_django/test_django/urls.py`中拆出来拆到每个app中各自管理，进行解耦。

路由管理通常具体的做法是这样的：

把路由到每个app的路由写在项目的主路由里，主路由就完成了他的使命。

然后各自的app管理自己的路由。

具体的做法是，引入了`include`函数，这样能使主路由匹配到自己的部分匹配项，把剩余的项交给子app去处理。

举个例子：

我们对test_django/test_django/urls.py进行改造：

```python
from django.contrib import admin
from django.urls import path, re_path, include


urlpatterns = [
    path('admin/', admin.site.urls),
    # include的字符串其实就是app目录名.下面的urls.py
    path('app01', include('test_app.urls'))
]
```

在test_app下新建urls.py文件：

```python
from django.urls import path
from .views.index import index
urlpatterns = [
    path('/index', index)
]
```

在test_app下新建views目录，在test_app/views下新建index.py文件：

```python
from django.shortcuts import HttpResponse
def index(request):
    return HttpResponse('我是index')
```

ok，run一下：

`http://127.0.0.1:8000/app01/index`

我们发现可以正常工作。搞定了。

注意哈，django推荐在路由分发时，把分隔符`/`写在上一级路由的末尾，而不是下一级的开头。所以我们按照django推荐的方式写代码就变成了：

test_django/test_django/urls.py

```python
from django.contrib import admin
from django.urls import path, re_path, include


urlpatterns = [
    path('admin/', admin.site.urls),
    # include的字符串其实就是app目录名.下面的urls.py
    path('app01/', include('test_app.urls'))
]
```

urls.py

```python
from django.urls import path
from .views.index import index
urlpatterns = [
    path('index', index)
]
```



### 别名反向生成URL

#### 作用

先说说这玩意儿有啥用。

- reverse可以把提取的URL按照要求进行替换，计算得到响应所需要的新的URL。

- 运用别名反向生成真正能做到实现URL的软编码，而非硬编码。

- 运用别名可以提高可读性，使URL自带翻译。
- 在django权限管理时特别好用

试想一个场景，如果某个URL对应的视图函数的功能是：先做一番数据处理，然后展示另外一个页面，你怎么操作？

按照通常的思路，处理完之后，redirect到另一个页面的url即可。

ok，你实现了功能。过了几天，需求改了，上司觉得url命名不好，给你建议了个新的名字。

你要怎么快速改掉url? 难道用全局替换吗？ 累傻小子呢。。。

一个通用的思路是啥？不要重复写url。学c语言入门的时候老师就教了，对于这种常量，可以用define把它弄成一个符号，这样只需要一次替换。

ok, 你完全可以把URL写在一个配置文件里，然后如果有改动URL的需求，则只需要改配置文件就好了。这比累傻小子要强一点点。

然而问题来了，动态路由你怎么用URL配置文件进行配置？是不是还要自己写代码来做到匹配动态路由？

是的，没有错。但是不要重复造轮子，django已经给你写好了这种功能了。只要输入别名和参数，自动生成url. (当然，前提是生成的URL要符合你写的url的规则，否则生成失败)。这就是reverse函数。

#### 注意事项

**需要注意的是，反向生成的url，也是url. 它必须遵循你自己写的规则。**

比如  你的re_path是这么写的：

`re_path('/index([1-3]{2})', index, name='tmp1')`

然后reverse是这么写的：

```python
def index(request, tst):
    rev = reverse('tmp1', args=(22,))
    print(rev)
    return HttpResponse(rev)
```

那么，你反向生成的别名为tmp1的url，必须要满足你自己写的规则。即以/index为开头，后面跟着2个1-3的数字，否则报错。

另外啊，反向生成的url是一个完整的相对路径url。即，是从你的主路由开始的url。

上面的例子，生成的url就是`/app01/index22`

由于这个注意事项，所以跟动态路由的规则一样，遇到`()`就代表遇到一个参数，需要以元组的形式放在args中。

当然啦，如果URL的规则是命名的，即`(?P<tmp>[1-3]{3})`这种形式，那么需要以字典的形式传过去。

比如这是我们的规则：

```python
re_path('/index(?P<tmp>[1-3]{2})', index, name='tmp1')
```

那我们的视图函数就应该这么写：

```python
from django.shortcuts import HttpResponse
from django.urls import reverse
def index(request, tmp):
    rev = reverse('tmp1', kwargs={
        'tmp': '21'
    })
    print(rev)
    return HttpResponse(rev)
```

用这种字典的传参方式。



**另外，在django模板里也能用url的别名， 这大大提升了改需求的便捷性。**

怎么用？  会在模板章节详细说，这里只摆个例子。

`{% url "tmp1" %}`

这里是以上面的tmp1为别名的例子来写的。

这就做到了在一个地方修改，别的地方都用软编码即可。

咦？如果碰到动态路由怎么办？这样用别名怎么知道动态路由的url到底是什么呢？

好问题，可以以参数的形式给到template， 然后再向上面的代码传参即可。

即：

`{% url "tmp1" i %}`

这就是传好了参数。多个参数咋整？挨个传呗。

#### 用法

其实在注意事项里，已经写过了re_path的用法。

所以在这儿偷个懒，只写path的用法就行了。

path的用法是已经命名了的。所以直接套用re_path的视图函数的写法就一点问题都没有。

这是视图函数：

```python
from django.shortcuts import HttpResponse
from django.urls import reverse
def index(request, age):
    rev = reverse('tmp1', kwargs={
        'age': 2122222
    })
    print(rev)
    return HttpResponse(rev)
```

这是urls.py中的内容：

```python
from django.urls import path,re_path
from .views.index import index
urlpatterns = [
    path('/index/<int:age>', index, name='tmp1')
]
```

相信对照re_path很快就能看懂。

#### 本质

一定要记得反向生成URL的本质，方便，方便，方便。思路就是C语言的define定义一个全局的量。只不过django都帮我们做好了，包括动态路由之类的操作。





## DjangoORM

### ORM的功能

1. 操作表 - (创建表、修改表、删除表)， 其中修改表包括修改表结构、表数据类型。
2. 操作数据行 - (增删改查)

### DjangoORM的特点

- 没有提供链接mysql的能力，故需要用三方库(MySQLdb, pymysql等)连接数据库。我们一般用pymysql。它比MySQLdb(django默认连接的第三方连数据库的工具)好用很多。
-  djangoORM只能操作表和操作数据行，并没有操作数据库的能力。所以我们要自己手动创建好数据库让djangoORM去连接。

### DjangoORM初始化步骤

1. 在工程目录，本例中即为`test_django/test_django/`下的`__init__.py`文件中，写入以下代码：

   ```python
   import pymysql
   # 此举是设置一个虚假的pymysql版本号，骗过django，因为django3.x不能用较低版本的pymysql了
   pymysql.version_info = (1, 4, 13, "final", 0)
   pymysql.install_as_MySQLdb()
   ```

   此举是用pymysql为django提供连接数据库的能力。

2. 创建数据库 - (navicat和命令行创建均可)。

3. 配置数据库到django的项目配置文件settings.py中。

   ```python
   DATABASES = {
       'default': {
           'ENGINE': 'django.db.backends.mysql',
           'NAME': 'test_django', # 刚创建的数据库名称
           'USER': 'root', # 用户名
           'PASSWORD': '', # 密码
           'HOST': '', # mysql的部署机器ip, 默认为localhost
           'PORT': '' # mysql的端口号，默认为3306
       }
   }
   ```

到此，djangoORM的初始化已经搞定了。

### DjangoORM的相关概念

Django的ORM, 其本质上是用操作类和对象的方式来操作数据库。

在ORM的世界里，类对应着某张数据表；对象对应着表中的某条记录。

外键通常是设置在"一对多"关系中"多"的一边。在django的orm中，表示外键的成员对应的不是一个字段，而是一条记录(即对应着"一"中的一条记录)。但在数据库表中会生成一个外键对应的字段。

### DjangoORM建单表步骤                                                                                                                                                                                                                                                                                                                 

1. 创建一个类，此类必须继承自models.Model类。

   1. 类中的成员变量代表了表中的字段。
   2. 类中的外键指向另外一个类的对象。
   3. 下面是代码。代码中给出了一些djangoORM的规则。
      1. 如果不设置主键,DjangoORM会自动新增一列id, 自增，默认为主键。
      2. 对于CharField来说，max_length参数是必须有的。
   
   ```python
   from django.db import models
   
   class UserInfo(models.Model):
       # 显式指定了主键
       nid = models.BigAutoField(primary_key=True)
    # 相当于数据库中的varchar, 所以必须制定max_length
       username = models.CharField(max_length=32)
       password = models.CharField(max_length=64)
   ```
   
2. 去settings.py中将创建类的那个app注册在settings.py中。

   ```python
   INSTALLED_APPS = [
       'django.contrib.admin',
       'django.contrib.auth',
       'django.contrib.contenttypes',
       'django.contrib.sessions',
       'django.contrib.messages',
       'django.contrib.staticfiles',
       'test_app' # 我们新增的app
   ]
   ```

3. 创建数据表

   1. 创建迁移文件

      ```python
      python manage.py makemigrations
      ```

      此举会在已经注册过的app的目录下生成migrations目录，其中存放的就是迁移文件。

      迁移文件是啥东西呢？

      **其实迁移文件就是方便我们的工程快速移动的。它记录了我们创建表、更改表字段的全过程。别人拿到这个迁移文件之后，只要执行migrate操作就能立刻创建出表结构。**

   2. 执行迁移实际操作

      `python manage.py migrate`

   执行完了这两个步骤以后，我们的数据库表就建好了。可以用navicat看一下。

   表名为app名_类小写名的即为我们自己写的表。其余的表是django免费赠送的。

### DjangoORM单表修改表结构步骤

我们的表结构并不是一成不变的。随着业务的改动，我们的表结构有可能也会跟着改动。

修改表结构主要包含下面几种操作：

- 新增表字段
- 修改表字段
- 删除表字段

#### 新增表字段

django在数据库迁移时，是不知道你数据库表中有没有数据的。所以在新增表字段的时候，它就会很疑惑要给之前存在的数据行(即记录)增加的新字段搞个什么样的值。

**故，我们在进行新增表字段时，一定要制定一个default值。**

示例：

test_app/models.py

```python
from django.db import models

class UserInfo(models.Model):
    nid = models.BigAutoField(primary_key=True)
    username = models.CharField(max_length=32)
    password = models.CharField(max_length=64)

    # 新增表字段示例
    test_field = models.CharField(max_length=11, default='')
```

然后重新执行`python manage.py makemigrations` 和 `python manage.py migrate`即可。

注意哈，改了表之后，都要执行一遍这两条命令，后面不再赘述。

#### 修改表字段

修改表字段包含两种情况。一种是给表字段改个名字，定义不变。

一种是给表字段改个定义，名字不变。

咦？你问我为什么不是两个都可以改变？？那不就成新增了么23333

##### 给表字段改个定义，名字不变

```python
from django.db import models

class UserInfo(models.Model):
    nid = models.BigAutoField(primary_key=True)
    username = models.CharField(max_length=32)
    password = models.CharField(max_length=64)

    # 修改表字段定义
    test_field = models.IntegerField()
```

然后就正常生成表就好了。

##### 给表字段改名字，定义不变

```python
from django.db import models

class UserInfo(models.Model):
    nid = models.BigAutoField(primary_key=True)
    username = models.CharField(max_length=32)
    password = models.CharField(max_length=64)

    # 修改表字段名字
    test_field1 = models.IntegerField()
```

并没有啥特别的，直接生成即可。

#### 删除表字段

```python
from django.db import models

class UserInfo(models.Model):
    nnid = models.BigAutoField(primary_key=True)
    username = models.CharField(max_length=32)
    password = models.CharField(max_length=64)
```

直接删除即可。

### DjangoORM单表CRUD操作

#### 增

增加有**两种**方式。

一种是创建一行临时数据，最后保存的方式。(一行数据是一个对象，所以猜也能猜到怎么做) 。在创建对象的时候用参数的方式指定值就可以了。注意哈，这个方式为啥要用`tmp.save()`呢？因为这只是创建了一个对象，你得告诉ORM我修改完了你保存吧，怎么告诉？就是`tmp.save()`咯。

```python
from django.shortcuts import HttpResponse
from .. import models
def index(request):
    tmp = models.UserInfo(username='tetetete', password='tetetetetete')
    tmp.save()
    return HttpResponse('OK')
```



另外一种是直接create.在create中用关键字参数的形式指定参数值就可以了。

```python
from django.shortcuts import HttpResponse
from .. import models
def index(request):
    models.UserInfo.objects.create(username='dsadasd',password='dsadasdas')
    return HttpResponse('OK')
```

#### 删

删除分为范围删除和单个删除。不过操作上都是一样的。

我们只需要在查询出来的单个对象上执行delete()，或者在查询出来的querySet上执行delete()。

单个删除：

```python
from django.shortcuts import HttpResponse
from .. import models
def index(request):
    models.UserInfo.objects.get(nid=1).delete()
    return HttpResponse('OK')
```

多个删除：

```python
from django.shortcuts import HttpResponse
from .. import models
def index(request):
    models.UserInfo.objects.filter(nid__gt=1).delete()
    return HttpResponse('OK')
```

全部删除：

```python
from django.shortcuts import HttpResponse
from .. import models
def index(request):
    models.UserInfo.objects.all().delete()
    return HttpResponse('OK')
```

#### 改

修改的话和删除类似，也是有全部更新，部分更新和单个更新之分。

全部更新

```python
from django.shortcuts import HttpResponse
from .. import models
def index(request):
    models.UserInfo.objects.all().update(username='test')
    return HttpResponse('OK')
```

部分更新

```python
from django.shortcuts import HttpResponse
from .. import models
def index(request):
    models.UserInfo.objects.filter(username='test').update(username='11111')
    return HttpResponse('OK')
```

单个更新

```python
from django.shortcuts import HttpResponse
from .. import models
def index(request):
    models.UserInfo.objects.filter(nid=1).update(username='nid1')
    return HttpResponse('OK')
```

#### 查

查询语句太多了，这里只列举最常用的。

**非条件查询(把全表数据返回)**

```python
data = models.UserInfo.objects.all()
```

这里的data实际上是一个querySet对象，这是一个可迭代对象，我们可以认为这是个列表list。

它之中的元素是一个个的UserInfo对象(就是查询的是哪个类，就是哪个类的对象呗，因为对象代表一行数据嘛)。

这样的话会给我们带来处理上的麻烦，比如我们怎么转换成json数据传给前端？

好办，想让querySet中的元素不是对象，只需要这样写：

```python
data = models.UserInfo.objects.all().values()
```

这样元素就是变成了一个个的字典，方便我们进行序列化。

我们只需要把querySet转成list就可以方便的进行序列化了。

当然我们也可以把元素变成元组，但我们一般不这么做。为了知识结构完整放在这儿。

```python
data = models.UserInfo.objects.all().values_list()
```

**非条件查询(全表数据返回，但只返回若干列的值)**

```python
data = models.UserInfo.objects.all().values('username','nid')
```

只需要这样写就能筛选想要的列了。

**条件查询**

```python
from django.shortcuts import HttpResponse
from .. import models
def index(request):
    data = models.UserInfo.objects.values('username','nid').filter(nid__gt=2, nid__lt=5)
    print(list(data))
    return HttpResponse('OK')
```

我们只需要把查询条件写在filter里就好了。filter中对于等于，直接写等于即可。

对于大于，要在关键字参数名后面加双下划线__(神奇的双下划线)再加上gt(greater than)。小于是lt(lower than).

多个关键字参数之间默认是与(and)的约束关系。对于or之类的关系，以后会说。

注意哈，values和filter并没有先后次序之分。

**只查询一条数据(记录)**

```python
from django.shortcuts import HttpResponse
from .. import models
def index(request):
    data = models.UserInfo.objects.get(nid=1)
    return HttpResponse('OK')
```

get中指定的是查询条件。当然啦，我们只查一个当然用主键比较好。

get有啥用? 它一般用在要更新数据的时候，先查出来，查出来的是个对象，对这个对象修改数据之后save就完事儿了。

#### 总结

```python
UserInfo.objects.all() # 拿到全部对象
UserInfo.objects.filter(id=1, xx=2) # and 关系
UserInfo.objects.all().first() # 取第一个结果
UserInfo.objects.all().count() # 统计多少条数据
UserInfo.objects.all().update() # 更新
UserInfo.objects.all().delete() # 删除
UserInfo.objects.all()[1:11] # 范围
```



### DjangoORM多表操作之一对多建表

在**一**对**多**的表关系中，我们一般在**多**的一方设置一个**外键**。

如果django确定这是两张新表建立了外键关系，那么事情就简单了。

如果其中一张 or 两张表都不是新表，那么django就搞不清楚表中到底有没有数据，它需要给外键制定一个**默认值**，但是不能像普通字段一样用`default`指定。它必须用`null=True`来指定默认值为**null**. 



两张新表一对多示例：

```python
from django.db import models

# Create your models here.
class ClassInfo(models.Model):
    cid = models.BigAutoField(primary_key=True)
    cname = models.CharField(max_length=32)

class StuInfo(models.Model):
    sid = models.BigAutoField(primary_key=True)
    sname = models.CharField(max_length=32)
    CI = models.ForeignKey('ClassInfo', on_delete=models.CASCADE)
```

我们执行makemigrations 和 migrate后，去看看数据库：

我们会发现外键字段是CI_id。 没错这就是django帮我们干的事儿。CI_id用来存储外键。而代码里的CI，其实是一个对象，代表了ClassInfo的一行数据（即被StuInfo外键关联的数据）。

我们再来看看两张非新表建立一对多关系的示例：

```python
from django.db import models
# Create your models here.
class UserInfo(models.Model):
    user_id = models.BigAutoField(primary_key=True)
    user_name = models.CharField(max_length=32)
    ug = models.ForeignKey('UserGroup', on_delete=models.CASCADE, null=True)

class UserGroup(models.Model):
    group_id = models.BigAutoField(primary_key=True)
    group_name = models.CharField(max_length=32)
```

### DjangoORM多表操作之一对多写数据

```python
from django.shortcuts import HttpResponse
from django.views import View
from .models import UserGroup, UserInfo
# Create your views here.
class CBV(View):
    def get(self, request):
        UserInfo.objects.create(user_name='testvth', ug_id=1)
        return HttpResponse(200)
```

那么，用户哪知道ug_id应该传什么呢？

好问题。在这种新增的时候，特别是外键的时候，我们一般会返回`UserGroup`表中的所有内容供用户选择，用户选择的`UserGroup`中的主键就代表了`ug_id`，因为这两个值是相同的。

### DjangoORM多表操作之一对多查询

诶？我们怎么用django进行连表查询来着？？即join那一套？

没事djangoORM自带了这个功能。我们平时多表查询基本也都是用外键的方式，所以django提供了外键方式查询。

#### 正向查询(查"多"中的数据，顺带把对应的外键信息也查出来)

```python
from django.shortcuts import HttpResponse
from django.views import View
from .models import UserGroup, UserInfo
# Create your views here.
class CBV(View):
    def get(self, request):
        datas = UserInfo.objects.all()
        for item in datas:
            # 对象方式跨表
            print(item.user_id, item.user_name, item.ug_id, item.ug.group_name)
        return HttpResponse(200)
```

我们直接点，就能拿到UserGroup中的数据了，因为ug代表了UserGroup的一个对象(即一条记录)嘛。

**那不能像mysql一样直接就可以返回拼接好的数据嘛？这是查出来还要自己负责拼接？**

不用，哪有这么麻烦，别忘了还有神奇的双下划线。

```python
from django.shortcuts import HttpResponse
from django.views import View
from .models import UserGroup, UserInfo
# Create your views here.
class CBV(View):
    def get(self, request):
        # values方式跨表
        print(UserInfo.objects.all().values('user_name', 'ug__group_name'))
        return HttpResponse(200)
```

使用方法：只需要用`外键成员__对应的表中的成员名`就可以访问了。

#### 反向查询

反向查询即通过查询**一对多**中**一**的部分，把**跟他关联的多的部分的数据**都给查出来。

```python
from django.shortcuts import HttpResponse
from django.views import View
from .models import UserGroup, UserInfo
# Create your views here.
class CBV(View):
    def get(self, request):
        datas = UserGroup.objects.all()
        data = datas.first()
        # 对象方式跨表
        print(data.userinfo_set.values())
        return HttpResponse(200)
```

总结一下，就是`对象.关联表小写+下划线+set`相当于是一个querySet，其中的元素是与本对象有外键关联的UserInfo对象。

用values跨表：

```python
from django.shortcuts import HttpResponse
from django.views import View
from .models import UserGroup, UserInfo
# Create your views here.
class CBV(View):
    def get(self, request):
        print(UserGroup.objects.values('group_name', 'userinfo__user_name'))
        return HttpResponse(200)
```

即跨表时使用的是`另外一个表的小写表名__字段`

#### 跨表总结

django中的一对多跨表，其实就是个**left join**。 谁在前面谁就是left(以谁为基准)。



### DjangoORM补充

#### 排序

```python
from django.shortcuts import HttpResponse
from django.views import View
from .models import UserGroup, UserInfo
# Create your views here.
class CBV(View):
    def get(self, request):
        # 直接在这儿
        # 按group_name正序
        print(UserGroup.objects.values('group_name').order_by('group_name'))
        # 按group_name逆序 - 注意一下逆序的写法
        print(UserGroup.objects.values('group_name').order_by('-group_name'))
        return HttpResponse(200)
```

#### 分组

`django.db.models`中有聚合函数，分别为`Count, Max, Min, Sum` 

分组要用`annotate`方法，即annotate方法相当于`goup_by`.这里`values`起到的作用是指定了按什么分组，这个例子是按`ug_id`进行分组的。`x`相当于是给`count`这个聚合函数查出来的结果起了个别名。

```python
from django.shortcuts import HttpResponse
from django.views import View
from .models import UserGroup, UserInfo
from django.db.models import Count, Max, Min, Sum
# Create your views here.
class CBV(View):
    def get(self, request):
        # 
        print(UserInfo.objects.values('ug_id').annotate(x=Count('user_id')))
        return HttpResponse(200)
```

补充一点，我们写的`djangoORM`，可以让他展现出生成的`sql`语句是啥样的，只需要在后面点出来`query`属性。

比如这样：

```python
from django.shortcuts import HttpResponse
from django.views import View
from .models import UserGroup, UserInfo
from django.db.models import Count, Max, Min, Sum
# Create your views here.
class CBV(View):
    def get(self, request):
        print(UserInfo.objects.values('ug_id').annotate(x=Count('user_id')).query)
        return HttpResponse(200)
```



另外再补充一点，在`djangoORM`的世界里，`filter()`被当成了`where`和`having`来使用。具体是被当成啥，要看用`filter`的时机。 `filter`用在`annotate`之前(或者语句中根本没有出现`annotate`)， 则是`where`。用在`annotate`之后，则是`having`。

```python
from django.shortcuts import HttpResponse
from django.views import View
from .models import UserGroup, UserInfo
from django.db.models import Count, Max, Min, Sum
# Create your views here.
class CBV(View):
    def get(self, request):
        # 相当于having了。
        print(UserInfo.objects.values('ug_id').annotate(x=Count(
            'user_id')).filter(x__gt=1))
        return HttpResponse(200)
```

#### filter详解

```python
from django.shortcuts import HttpResponse
from django.views import View
from .models import UserGroup, UserInfo
from django.db.models import Count, Max, Min, Sum
# Create your views here.
class CBV(View):
    def get(self, request):
        # 大于1
        print(UserInfo.objects.filter(user_id__gt=1))
        # 小于1
        print(UserInfo.objects.filter(user_id__lt=1))
        # 小于等于1
        print(UserInfo.objects.filter(user_id__lte=1))
        # 大于等于1
        print(UserInfo.objects.filter(user_id__gte=1))
        # 等于1或者2或者3均可
        print(UserInfo.objects.filter(user_id__in=[1,2,3]))
        # 大于等于1小于等于2
        print(UserInfo.objects.filter(user_id__range=[1,2]))
        # 查找以test开头的
        print(UserInfo.objects.filter(user_name__startswith='test'))
        # 查找以vth结尾的
        print(UserInfo.objects.filter(user_name__endswith='vth'))
        # 查找包含P的
        print(UserInfo.objects.filter(user_name__contains='P'))
        # 对filter取反(意思就是取出所有不符合条件的)
        print(UserInfo.objects.exclude(user_name__contains='P'))
        return HttpResponse(200)
```

#### F

试想一下，我们如果要对所有人或者是部分人的年龄+1, 怎么破？

如果是单个人，根据主键id查出来这个对象，对对象修改然后save就行了。

如果多个人呢？我们能这么写吗？循环不累吗？python性能很高吗能扛得住这么写吗？

灵机一动：

```python
from django.shortcuts import HttpResponse
from django.views import View
from .models import UserGroup, UserInfo
from django.db.models import Count, Max, Min, Sum, F
# Create your views here.
class CBV(View):
    def get(self, request):
        UserInfo.objects.all().update(user_age=user_age+1)
        
```

这样是行不通的，user_age是个参数的命名传递，当前方法下面没有这玩意儿啊。

现在问题转化成了，我们要怎么才能让'user_age'是数据库里的'user_age'呢，换句话说，要怎么样才能合法的使用这样的语法呢？

怎么才能如我所愿？F!

```python
from django.shortcuts import HttpResponse
from django.views import View
from .models import UserGroup, UserInfo
from django.db.models import Count, Max, Min, Sum, F
# Create your views here.
class CBV(View):
    def get(self, request):
        # 解决方案
        print(UserInfo.objects.all().update(user_age=F('user_age')+1))
        return HttpResponse(200)
```

注意，F也是有弱点的。**F当前只能用加减乘除取余等运算**。换句话说，它只适合处理数字型的值，处理其他值会报错。

#### Q

Q是用于构造复杂的查询条件的。

前面我们说过filter的特点，多个参数之间的关系被写死了是and关系，Q就是用来解决这个问题(即复杂的查询条件的问题)的。

Q的简单使用：

```python
from django.shortcuts import HttpResponse
from django.views import View
from .models import UserGroup, UserInfo
from django.db.models import Count, Max, Min, Sum, F, Q
# Create your views here.
class CBV(View):
    def get(self, request):
        # 相当于没写Q
        print(UserInfo.objects.filter(Q(user_id=1)).values())
        # 通过Q的封装，变成了或
        print(UserInfo.objects.filter(Q(user_id=1) | Q(user_name__endswith='cxz')).values())
        # 通过Q的封装，变成了与(脱裤子放屁，直接写filter里更香)
        print(UserInfo.objects.filter(Q(user_id=1) & Q(user_name__endswith='cxz')).values())
        return HttpResponse(200)
```

如果想这样拼装，没有函数式编程经验的你可能会懵逼，Q还提供了另外一种拼装方式。

```python
from django.shortcuts import HttpResponse
from django.views import View
from .models import UserGroup, UserInfo
from django.db.models import Count, Max, Min, Sum, F, Q
# Create your views here.
class CBV(View):
    def get(self, request):
        # Q的组合容器。可以认为Q是个递归似的玩意儿
        condition = Q()

        # q1
        q1 = Q()
        # 设置q1的内部连接规则，这里选择的是OR
        q1.connector = q1.OR
        # 添加的元素是一个个的元组
        q1.children.append(('user_id__gt', 1))
        q1.children.append(('user_id__lte',1))

        q2 = Q()
        # 设置q2的内部连接规则，这里选择的是AND
        q2.connector = Q.AND
        q2.children.append(('user_name__startswith', 'vth'))
        q2.children.append(('user_name__endswith', 'cxz'))

        # 设置q1和q2的连接规则
        # Q.AND是把q1当成一个整体
        condition.add(q1, Q.AND)
        # Q.OR代表了q2这个整体和q1的关系
        condition.add(q2, Q.OR)

        print(UserInfo.objects.filter(condition).query)
        return HttpResponse(200)
```

当然啦，对于高层封装的Q，其中元素也为Q的对象的时候，尽量都用and，这样统一起来比较好。

## Django视图

django的视图分为CBV和FBV。

所谓的FBV是指处理业务逻辑的是函数，而CBV就是说我们把业务逻辑写在了类中。

### FBV

```python
# test_app/urls.py
from django.urls import path
from .views.index import index
urlpatterns = [
    path('index/2', index)
]
```

```python
# test_app/views/index.py
from django.shortcuts import HttpResponse
from .. import models
def index(request):
    models.UserInfo.objects.filter(nid=1).update(username='nid1')
    return HttpResponse('OK')
```

我们可以看到，此时写在path中的是个视图函数。而且我们的实现方式是在index.py中直接定义的一个函数。

### CBV

1. CBV在实现时，在视图函数所在的文件中应写成一个类的形式，该类需要继承自`django.views.View`类。
2. 在这个类中要实现`HTTP`协议的方法对应的小写的方法。比如处理`get`请求的方法我们也要定义为`get`.
3. 在`urls.py`中的`path`里，不能直接写类名，要写`类名.as_view()`
4. HTTP协议支持的方法：'get', 'post', 'put', 'patch', 'delete', 'head', 'options', 'trace'
5. View中的逻辑是这样的，一执行as_view()方法，它会返回一个叫view的函数。这个函数会调用dispatch方法，而dispatch方法会通过反射的形式，调用我们写的业务逻辑，然后接收我们业务逻辑的返回值，然后dispatch、view一层层往上返回，最终返回给前端。

```python
# test_app/urls.py
from django.urls import path
from .views.index import CBVTest
urlpatterns = [
    path('index/2', CBVTest.as_view())
]
```

```python
# test_app/views/index.py
from django.shortcuts import HttpResponse
from django.views import View
from .. import models

class CBVTest(View):
    def get(self,request):
        return HttpResponse('get completed.')
```

既然业务逻辑的入口和出口都在dispatch方法，那么我们可以重写dispatch方法来做到类似于装饰器的功能。

```python
from django.shortcuts import HttpResponse
from django.views import View
from .. import models

class CBVTest(View):
    def get(self,request):
        return HttpResponse('get completed.')

    def dispatch(self, request, *args, **kwargs):
        # before exec to do something
        res = super(CBVTest, self).dispatch(request, *args, **kwargs)
        # after exec to do something
        return res
```

### Django分页

占个坑，目前用不到，写完论文再学。