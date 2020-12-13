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