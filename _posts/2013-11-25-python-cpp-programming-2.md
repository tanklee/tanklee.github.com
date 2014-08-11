---
layout: post
title : C++和Python混合编程 - python嵌入C++
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


##一、Python C API写 python embedding

python嵌入 C/C++ 的程序，我们需要在一条 #include 语句处含入 Python.h，并在程序开始时调用 Py_Initialize()，在程序结束时调用 Py_Finalize()。

    #include <Python.h>
    // using python c API to wrte a python embedding

    const static char* python_code = "\
    from time import time,ctime\n\
    print 'Today is', ctime(time())\n";

    int main()
    {  
        Py_Initialize();
       
        PyRun_SimpleString(python_code);
       
        Py_Finalize();
        return 0;
    }

编译：

    [tanli@p04bc boostpython]$g++ sampleEmbed.cpp -I /usr/include/python2.7 -lpython2.7 -o sampleEmbed
    [tanli@p04bc boostpython]$./sampleEmbed
    Today is Fri Nov 22 10:18:50 2013

##二、boost.python 写 python embedding

boost.python 写python embedding更简单些,exec执行一个语句，exec_file执行一个python程序

    #include <iostream>
    #include <boost/python.hpp>
    using namespace std;
    using namespace boost::python;

    int main(int argc, char *argv[])
    {
      Py_Initialize();

      object main_module = import("__main__");
      object main_namespace = main_module.attr("__dict__");
      exec("hello = file('hello.txt', 'w')\n"
           "hello.write('Hello world!')\n"
           "hello.close()",
           main_namespace);
      exec("result = 5 ** 2", main_namespace);
      int five_squared = extract<int>(main_namespace["result"]);
      cout << "The five_squeared caculated by python is " << five_squared << endl;

       // Load the sys module.
      object sys = import("sys");

      // Extract the python version.
      std::string version = extract<std::string>(sys.attr("version"));
      std::cout << version << std::endl;


      object simple = exec_file("simple.py",main_namespace, main_namespace);
      object add = main_namespace["add"];
      int val = extract<int>(add(5, 100));
      cout << "Python has caculated add as " << val << endl;


      return 0;
    }

编译运行下：

    [tanli@p04bc boostpython]$./embeddByBooostPy
    The five_squeared caculated by python is 25
    2.7 (r27:82500, Oct 20 2010, 13:31:35)
    [GCC 3.4.5 20051201 (Red Hat 3.4.5-2)]
    Python has caculated add as 105

