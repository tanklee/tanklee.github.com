---
layout: post
title : 玩玩Django - 环境搭建
description : 项目中一些工具需要提供web接口，所以抽时间看了看web开发, 好久之前在学校的时候接触过ASP, PHP ，但感觉太过繁琐，要自己设计前台页面和又要处理后台逻辑然后又代码又混在一起。显然，对于我们这种非专业web开发的人员的快速的开发需求，还有需要借组强大web开发框架的。看了下框架很多很多，有基于PHP的，也有基于python的。Diaogo就是这么一个很火的Python web 框架，之前的项目组用到的开源工具Review board, OSQA都是基于Diango开发的，自己也凭着感觉简单改过些源码，看来是时候看看Diango了，那么就从环境搭建开始吧。
category : Python
tags : [Python]
---
{% include JB/setup %}

##一、环境配置

（1）安装Python
window比较简单，linux默认已经安装，我这里的版本Python2.7.6,有了Python就可以安装Django了

（2）下载最新版本Diango.1.6.0，
https://www.djangoproject.com/download/

（3）—解压进入目录然后执行:

	$ python setup.py install

（4）检测是否安装成功:
	$ python
	>>> import django
	>>> print django.VERSION
	(1, 6, 0, 'final', 0)
	>>> print django.__file__
	E:\Python27\lib\site-packages\django\__init__.pyc

（5）设置环境变量（可选）
Django安装成功后默认位于 $PYTHON_INSTLL_HOME/lib/site-packges/django为方便使用Django提供的一些命令行工具，加入$PYTHON_INSTLL_HOME/lib/site-packges/django/bin到$PATH（通过设置环境变量)


##二、第一个Django程序Hello world

写个Django 版本的hello world吧

（1）创建一个名字为mysite的Django项目

	$ E:\toycode\django-project>django-admin.py startproject mysite

Diango会为你创建一个项目的目录，结构如下：

	mysite/
	     manage.py           	#对项目操作的命令
	     mysite/                      
	           __init__.py     	#这个目录是个python包
	          setting.py       	#项目设置文件
	          urls.py           #URL映射管理
	          wsgi.py           #Python应用程序或框架和Web服务器之间的一种接口

（2）利用Diango内置的web服务器快速调试网页，默认开8000端口

	E:\toycode\django-project>cd mysite
	E:\toycode\django-project\mysite>manage.py runserver
	Validating models...

	0 errors found
	November 27, 2013 - 09:54:27
	Django version 1.6, using settings 'mysite.settings'
	Starting development server at http://127.0.0.1:8000/
	Quit the server with CTRL-BREAK.
	[27/Nov/2013 09:55:04] "GET / HTTP/1.1" 200 1757

（3）预览网页
浏览器中输入 localhost:8000, 你将看到欢迎页面:

	It worked!
	Congratulations on your first Django-powered page.

	Of course, you haven't actually done any work yet. Next, start your first app by running python manage.py startapp [appname].
	You're seeing this message because you have DEBUG = True in your Django settings file and you haven't configured any URLs. Get to work!


（4）增加自己的页面
新建一个文件 views.py 在 mysite/mysite 目录

	from django.http import HttpResponse

	def hello(request):
	    return HttpResponse("Hello world")


（5）增加URL 映射
修改文件 mysite/mysite/url.py

	urlpatterns = patterns('',
	    # Examples:
	    # url(r'^$', 'mysite.views.home', name='home'),
	    # url(r'^blog/', include('blog.urls')),

	    url(r'^admin/', include(admin.site.urls)),
	    url('^hello/$','mysite.views.hello'),
	)

（6）在浏览器中预览我们自己的页面

输入localhost/hello, 你将看到:
Hello World!
		