---
layout: post
title : C++和Python混合编程 - C++写python扩展
description : 早就听说过了对于复杂的系统一般会结合多种语言进行开发，最近摸索了下C++和Python混合编程，在此总结一下，做为笔记。 
category : C/C++
tags : [C/C++, Python, Boost]
---
{% include JB/setup %}

对于C++和python混合编程实际上包含两部分内容：
* 用C++写python扩展（extending）: 即所谓的在python中调用C/C++代码，一般用于对效率要求高核心模块用C/C++编写，通过一些处理后生成动态库如cppmod.so,然在python代码中import cppmod，即可调用C/C++
* python脚本嵌入到C++程序中(embedding): 即所谓的C++代码调用python,C++ app要嵌入python解析器从而调用python代码

就实现而言，对于这两者都可以直接用Python/C API,具体可以去参考python文档，但都比较麻烦

* 对于1， 主要的方案有 boost.python 以及swig, 后者不太了解，本篇主要讲利用boost.python，对C/C++处理后生成动态库，python导入后调用C/C++代码
* 对于2， C/C++代码中嵌入python,也可以用 boost.python，当然直接 Python/C API也可以


##一、安装boost(latest version boost1.55.0)

    [tanli@p04bc boost_1_55_0]$./bootstrap.sh --with-python=/usr/bin/python2.7 --with-python-root=/usr --with-python-version=2.7
    [tanli@p04bc boost_1_55_0]$./b2 variant=debug link=static threading=multi --with-python

这里为节省时间，只编译boost.python库的static-debug-multi版本，默认在当前目录构建，生成的静库文件位于./statge，
构建完成好了看到信息如下：

    The Boost C++ Libraries were successfully built!

    The following directory should be added to compiler include paths:

        /home/tanli/soft/boost_1_55_0

    The following directory should be added to linker library paths:

        /home/tanli/soft/boost_1_55_0/stage/lib

注意: boost.python 库是和python版本相关的，当安装Boost, 会自动探测Python版本, 如果有多个python版本，还是需要指定的，否则后面出现一些错误。


##二、用boost.python库导出C++函数

在cpp文件中增加导出接口代码

    #include <iostream>
    #include <boost/python.hpp>
    using namespace boost::python;

    void SayHello()
    {
        std::cout<<"hello, world!"<<std::endl;
    }

    void SayBye()
    {
        std::cout<<"good bye!"<<std::endl;
    }

    BOOST_PYTHON_MODULE(hello)   // generate python module hello
    {
        def("SayHello", SayHello);
        def("SayBye", SayBye);
    }

编译成动态库:

    [tanli@p04bc boost_1_55_0]$ g++ hello.cpp -shared -fPIC -o hello.so -I ~/soft/boost_1_55_0 -L ~/soft/boost_1_55_0/stage/lib -lboost_python -I /usr/include/python2.7
    /usr/bin/ld: /home/tanli/soft/boost_1_55_0/stage/lib/libboost_python.a(registry.o): relocation R_X86_64_32 against `a local symbol' can not be used when making a shared object; recompile with -fPIC
    /home/tanli/soft/boost_1_55_0/stage/lib/libboost_python.a: could not read symbols: Bad value
    collect2: ld returned 1 exit status

原来是boost.python静态库有问题，要加编译选项-fPIC, 看来得改编译选项，google了下，可以怎么改
针对boost_1_55_0版本，修改tools/build/v2/tools/gcc.jam，注释掉391行：
    
    #if $(link) = shared

按照上面命令重新编译boost.python库，然后在编译上面程序，这回好了,再打开python导入动态库调用吧

    [tanli@p04bc boostpython]$g++ hello.cpp -shared -fPIC -o hello.so -I ~/soft/boost_1_55_0 -L ~/soft/boost_1_55_0/stage/lib -lboost_python -I /usr/include/python2.7
    [tanli@p04bc boostpython]$python
    Python 2.7 (r27:82500, Oct 20 2010, 13:31:35)
    [GCC 3.4.5 20051201 (Red Hat 3.4.5-2)] on linux2
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import hello
    >>> hello.SayHello()
    hello, world!
    >>> hello.SayBye()
    good bye!
    >>> exit()


##三、用boost.python库导出C++类

刚才的例子是导出一个函数给python用，那么对于class呢？

    #include <string>
    #include <boost/python.hpp>
    #include <boost/python/module.hpp>
    #include <boost/python/def.hpp>
    using namespace std;

    class CHello{
    public:
        string GetString()
        {
            return "CHello::GetString()";
        }

        int GetInt()
        {
            return 100;
        }
    };


    BOOST_PYTHON_MODULE(hello_class)
    {
        using namespace boost::python;
        class_<CHello>("CHello")
          .def("GetString",&CHello::GetString)
          .def("GetInt",&CHello::GetInt)
        ;
    }


编译运行：

    [tanli@p04bc boostpython]$ g++ helloclass.cpp -shared -fPIC -o hello_class.so -I ~/soft/boost_1_55_0 -L ~/soft/boost_1_55_0/stage/lib -lboost_python -I /usr/include/python2.7
    [tanli@p04bc boostpython]$python
    Python 2.7 (r27:82500, Oct 20 2010, 13:31:35)
    [GCC 3.4.5 20051201 (Red Hat 3.4.5-2)] on linux2
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import hello_class
    >>> a = hello_class.CHello()
    >>> a.Get
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    AttributeError: 'CHello' object has no attribute 'Get'
    >>> a.GetString()
    'CHello::GetString()'
    >>> a.GetInt()  
    100
    >>> quit()


##四、用boost.python库导出构造函数

对于非默认构造函数怎么导出呢：

    #include <string>
    #include <boost/python.hpp>
    using namespace std;

    class CHello{
    public:
        CHello(string msg)
            : m_msg(msg)
        {
            m_id = 0;
        }

        CHello(int id)
        {
            m_id = id;
        }

        string GetString() const
        {
            return m_msg;
        }
        string SetString(string msg)
        {
            m_msg = msg;
        }
        int GetInt()
        {
            return m_id;
        }
        int SetInt(int id)
        {
            m_id = id;
        }

        string m_msg;
        int m_id;
    };


    BOOST_PYTHON_MODULE(hello_class)
    {
        using namespace boost::python;
        class_<CHello>("CHello", init<string>())
          .def(init<int>())
          .def("SetString", &CHello::SetString)
          .def("GetString", &CHello::GetString)
          .def("GetInt", &CHello::GetInt)
        ;
    }


测试结果：

    [tanli@p04bc boostpython]$g++ helloclass.cpp -shared -fPIC -o hello_class.so -I ~/soft/boost_1_55_0 -L ~/soft/boost_1_55_0/stage/lib -lboost_python -I /usr/include/python2.7
    [tanli@p04bc boostpython]$python
    Python 2.7 (r27:82500, Oct 20 2010, 13:31:35)
    [GCC 3.4.5 20051201 (Red Hat 3.4.5-2)] on linux2
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import hello_class
    >>> a = hello_class.CHello()  
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    Boost.Python.ArgumentError: Python argument types in
        CHello.__init__(CHello)
    did not match C++ signature:
        __init__(_object*, int)
        __init__(_object*, std::string)
    >>> a = hello_class.CHello(1)
    >>> a.GetInt()
    1
    >>> b = hello_class.CHello("hello world!")
    >>> b.GetString()
    'hello world!'


##五、总结

总的来说就是在C++里引入导出接口代码，然后生成动态库，python里以导入模块的方式加载动态库，然后调用C++程序，更多复杂的导出，
比如继续，虚函数，还是参考手册吧，这里就不介绍了。

