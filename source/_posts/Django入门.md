---
title: Django学习笔记一初识Django
date: 2022-03-5
top: false
tags: Django
categories: 后端
---
# 1 Django简介

## 1.1 基本介绍

Django 是一个由 Python 编写的一个开放源代码的 Web 应用框架。开发者使用Django，只要很少的代码，就可以轻松完成一个网站所需要的大部分内容，并进一步开发出全功能的 Web 服务 。Django 本身基于 MVC 模型，即 Model（模型）+ View（视图）+ Controller（控制器）设计模式，MVC 模式使后续对程序的修改和扩展简化，并且使程序某一部分的重复利用成为可能。

MVC 优势：

- 低耦合
- 开发快捷
- 部署方便
- 可重用性高
- 维护成本低
- ...

Python 加 Django 是快速开发、设计、部署网站的最佳组合。

## 1.2 开发框架模型

### 1.2.1 简介

目前的开发框架模型可以按是否前后端分离来划分。其中各自特点如下：

前后端不分离特点：

- 后端需控制数据的展示
- 前后端不分家，耦合严重
- 返回的是HTML页面，适应性、拓展性差
  - 只能用于浏览器，其它终端不匹配

前后端分离的特点：

- 当前主流
- 后端只对数据进行处理，只提供数据
- 前端效率、页面好不好看，全由前端负责，前后端完全独立
- 解耦合
- 前后端同时开发，缩小业务上线周期
- 绝大多数情况下，前端发送json格式的参数，后端同样以json格式的数据返回
  - 适应性、拓展性好
  - 适合多终端运行同一套接口（PC、APP、小程序等）

在本文中，对前后端不分离的开发框架模型这里不作叙述，这里主要介绍前后端分离的MVC模型和MVT模型。

### 1.2.2 MVC模型

- 体现：Java常用MVC模式，比如SpringMVC。
- MVC各部分的解释
  * M/Model（模型）：主要封装对数据层的操作，对数据库的数据进行增删改查操作。
  * V/View（视图）：用于封装结果的，然后生成用于展示html。
  * C/Controller（控制器）：用于接收请求，然后处理业务逻辑，并返回结果。它处于Model和View之间，与两者交互。

* 图解

  ![MVC模型](https://raw.githubusercontent.com/unique-pure/PicLibrary/main/img/13826085-1840aa6fc7949581.png)

### 1.2.3 MVT模型

* 体现：Python中的Django框架就是MVT
* MVT各部分的解释
  * M/Model（模型）：与MVC的M功能相同，主要封装对数据层的操作，对数据库的数据进行增删改查操作。
  * V/View（视图）：与MVC的C功能相同，接收请求，逻辑处理，返回结果。
  * T/Template（模板）：和MVC中的V功能相同，负责封装和生成要返回的html。

* 图解

  ![MVT模型](https://raw.githubusercontent.com/unique-pure/PicLibrary/main/img/13826085-528b41457070260b.png)

# 2 Django安装

如果你还未安装Python环境需要先下载Python安装包。注意：目前的 Django 1.6.x 以上版本已经完全兼容 Python 3.x。

使用`pip3 install Django==版本号 -i https://pypi.tuna.tsinghua.edu.cn/simple`即可安装Django，注意使用镜像源下载，不然下载速度会很慢。

检查是否安装成功：

![image-20220305175927603](https://raw.githubusercontent.com/unique-pure/PicLibrary/main/img/image-20220305175927603.png)

# 3 Django项目框架—创建第一个项目

## 3.1 Django管理工具

安装好Django之后，我们现在就有了可用的管理工具`django-admin`，这个其实就是一个`py`文件，代码如下：

```python
from django.core import management
 
if __name__ == "__main__":
    management.execute_from_command_line()
```

`django-admin.py`调用`django.core.management`来执行命令，这个函数会根据命令行参数解析出命令的名称，根据命令的名称来调用相应的Command执行命令。

## 3.2 创建第一个项目

使用`django-admin`来创建HelloWorld项目：

```django
django-admin startproject HelloWorld
```

创建完成后，我们可以查看HelloWorld的目录结构如下：

![image-20220305191934585](https://raw.githubusercontent.com/unique-pure/PicLibrary/main/img/image-20220305191934585.png)目录说明：

- **HelloWorld:** 项目的容器。
- **manage.py:** 一个实用的命令行工具，可让你以各种方式与该 Django 项目进行交互。
- **HelloWorld/__init__.py:** 一个空文件，告诉 Python 该目录是一个 Python 包。
- **HelloWorld/asgi.py:** 一个 ASGI 兼容的 Web 服务器的入口，以便运行你的项目。
- **HelloWorld/settings.py:** 该 Django 项目的设置/配置。
- **HelloWorld/urls.py:** 该 Django 项目的 URL 声明; 一份由 Django 驱动的网站"目录"。
- **HelloWorld/wsgi.py:** 一个 WSGI 兼容的 Web 服务器的入口，以便运行你的项目。

我们进入HelloWorld目录后输入以下命令启动服务器：

```django
python3 manage.py runserver 0.0.0.0:8000
```

其中`0.0.0.0`是为了让其他电脑能连接到开发服务器，8000 为端口号。如果不说明，那么端口号默认为 8000，但建议指明端口号。

在浏览器输入我们的ip地址（如果是云服务器，则输入公网地址，如果是本机，则可以输入本机地址：127.0.0.1）。这里特别说明，若是云服务器，我们必须在控制台开放8000端口号和80端口号，要不然访问不了。如果没有将IP地址添加到ALLOWED_HOSTS，则会出现如图错误：

![](https://raw.githubusercontent.com/unique-pure/PicLibrary/main/img/image-20220305194113293.png)

我们需要将IP地址添加到`settings.py`中的`ALLOWED_HOSTS`中。再次访问出现如下界面，此为Django的默认界面：

![](C:/Users/pursuit/AppData/Roaming/Typora/typora-user-images/image-20220305194448542.png)

## 3.3 视图和URL配置

在先前创建的HelloWorld目录下的HelloWorld目录新建一个views.py，并输入代码：

```python
from django.http import HttpResponse
 
def hello(request):
    return HttpResponse("Hello world ! ")
```

接着，绑定 URL 与视图函数。打开 urls.py 文件，删除原来代码，将以下代码复制粘贴到 urls.py 文件中：

```python
from django.urls import path
 
from .views import hello
 
urlpatterns = [
    path('', hello),
]
```

完成后，启动 Django 开发服务器，并在浏览器访问打开浏览器并访问：

![](https://raw.githubusercontent.com/unique-pure/PicLibrary/main/img/image-20220305200823175.png)我们可以修改path，例如改为`path('/hello', hello)`，那么url即为：`http://127.0.0.1/hello`。这个path函数可以接收四个参数，分别是两个必选参数：`route`、`view`和两个可选参数：`kwargs`、`name`。语法格式如下：

`path(route, view, kwargs=None, name=None)`

- route: 字符串，表示 URL 规则，与之匹配的 URL 会执行对应的第二个参数 view。
- view: 用于执行与正则表达式匹配的 URL 请求。
- kwargs: 视图使用的字典类型的参数。
- name: 用来反向获取 URL。

**注意：**项目中如果代码有改动，服务器会自动监测代码的改动并自动重新载入，所以如果你已经启动了服务器则不需手动重启。