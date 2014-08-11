---
layout: post
title : Django学习 - Template
description : 在前面的例子的中，我们会了简化view工作原理，直接在view函数中嵌入HTML代码，然后返回HttpResponse对象然而真实的web程序这样做很糟糕的， 想想该有多难维护呢，而且大web应用是页面设计人员和程序开发人员的分工协作的，这样搅合在一起， 还有什么灵活性和可维护性可言呢。 Django框架的设计的一个核心就是要松耦合， 即一个模块的改变应该不去影响另一个模块， 好了，就是这个道理，把页面设计的东西分离出来吧，这就是Django里面的template了。
category : Django
tags : [Django, Python]
---
{% include JB/setup %}

##一、模板库的使用
template是个单独的库，可以不在视图的直接使用，也不局限于html文件
,是一个基于文本的模板引擎

    F:\toycode\django-project\hello>python manage.py shell
    Python 2.7.3 (default, Apr 10 2012, 23:31:26) [MSC v.1500 32 bit (Intel)] on win
    32
    Type "help", "copyright", "credits" or "license" for more information.
    (InteractiveConsole)
    >>> from django import template
    >>> t = template.Template('My name is {{ name }}')
    >>> c = template.Context({'name': 'Tank'})
    >>> print t.render(c)
    My name is Tank
    >>> c = template.Context({'name': 'Hello'})
    >>> print t.render(c)
    My name is Hello

先创建一个 Template 对象，这个Template对象直接用页面显示字符串构造，里面包含一些变量
和标签，用作显示内容的占位符，然后用context 来传递显示内容，一个context是一系列变量
和它们值的集合，最后调用 Template 对象 的 render() 方法用context替换模板变量为真实的
值和执行块标签进行渲染页面。


##二、在视图中使用模板
在视图中使用模板, 改进方法1的硬编码，如下面代码方法2所示， 这样确实是用上了django 
的template，可是模板的内容还嵌入在Python代码中，显示和逻辑还是没有分开。于是我们抽
离出单独了模板文件，如方法3所示。这一页面显示已经完全抽离到模板文件了。

    from django.http import HttpResponse
    from django.template import Template, Context
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
       
        # method 1: hard code html
        #html = "<html><body>It is now %s.</body></html>" % now

        # method 2: using template
        #t = Template("<html><body>It is now {{ current_date }}.</body></html>")
        #html = t.render(Context({'current_date' : now}))

        # method 3: extract a template file
        fp = open('F:\\toycode\\django-project\\templates\\showdate.html')
        t = Template(fp.read())
        fp.close()
        html = t.render(Context({'current_date' : now}))

        return HttpResponse(html)

可是方法3也略显繁琐，每次都要检测文件，打开文件，关闭文件。一个框架这么可能对此视而不见，
于是django 具有模板自动载入功能，所谓自动载入，就是在配置文件中制定模板路径，我们编辑项目的
settings.py

    TEMPLATE_DIRS = (
        # Put strings here, like "/home/html/django_templates" or "C:/www/django/templates".
        # Always use forward slashes, even on Windows.
        # Don't forget to use absolute paths, not relative paths.
        'F:/toycode/django-project/mysite/mysite/templates',
        # os.path.join(os.path.dirname(__file__), 'templates').replace('\\','/'),
    )

然后直接调用get_tempalte方法，传入模板文件名称，这样我们就是可以使用该模板了，如方法4所示（1）创建一个名字为mysite的Django项目:

    from django.http import HttpResponse
    from django.template import Template, Context
    from django.template.loader import get_template
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
       
        # method 4: django template autoload and tempalte directory
        t = get_template('showdate.html')
        html = t.render(Context({'current_date' : now}))
        return HttpResponse(html)
        $ E:\toycode\django-project>django-admin.py startproject mysite

能不能再简单点， 如方法5所示，一行代码完成加载板板，渲染模板，返回HttpResponse

    from django.shortcuts import render_to_response
    import datetime
    def current_datetime(request):
        now = datetime.datetime.now()
        
        # method 5: load template and render shortcuts
        return render_to_response('showdate.html', {'current_date': now})
