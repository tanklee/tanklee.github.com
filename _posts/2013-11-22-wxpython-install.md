---
layout: post
title: "wxPython安装与环境配置"
description: "Python语言接触也有些时间，但平常主要是写写完成某些功能命令行的脚本。然后对于图形界面开发呢，考察了下，python的GUI toolkit主要有：内置的Tkinter库, pyQt, wxPython等。平时项目中wx用的比较多，那就探索下wxPython吧，正如名字一样，由开源的C++跨平台图形库wxWidgets，加上python语言绑定而成。结合python语言的简洁强大与wx的丰富的图形界面，我们可以快速的开发python GUI程序。"
category: wxWidgets
tags: [Python, wxWidgets]
---
{% include JB/setup %}

##一、Windows下安装

Windows下安装比较简单:
* 安装python(我这里是python2.7)
* 安装对应版本的wxPython(我的是wxPython2.8-win32-unicode-py27)

装好后wxpython出现在python包目录中E:\Python27\Lib\site-packages\wx-2.8-msw-unicode,用python 引入包import wx来写GUI程序，
差不多了，`python  demo.py`跑跑内带的demo看看，可以发现wxPython有丰富的ui 元素

    C:\Documents and Settings\tanli>python
    Python 2.7.6 (default, Nov 10 2013, 19:24:18) [MSC v.1500 32 bit (Intel)] on win
    32
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import wx
    >>> print wx.VERSION
    (2, 8, 12, 1, '')

##二、Linux下安装

Linux下要稍微复杂些：

(1)安装wxWigets 

下载wxGTK-2.8.12.tar.gz解压后在其目录下建立bld文件夹，然后运行configure：

    mkdir bld
    cd bld
    ../configure --prefix=/opt/wx/2.8 \
                 --with-gtk \
                 --with-gnomeprint 
                 --with-opengl 
                 --enable-debug 
                 --enable-geometry 
                 --enable-graphics_ctx 
                 --enable-sound --with-sdl 
                 --enable-mediactrl 
                 --enable-display 
                 --disable-debugreport 
                 --enable-unicode 
                 --with-libjpeg=builtin 
                 --with-libpng=builtin 
                 --with-libtiff=builtin 
                 --with-zlib=builtin 

(--enable-mediactrl ,可能有个链接错误，可以去掉)

(2)编译安装：

制作一个脚本，内容如下：

    make $* \
        && make -C contrib/src/gizmos $* \ 
        && make -C contrib/src/stc $* \

命名为 .make ， 放到bld目录下
然后运行：

    .make
    .make install
 
(3)安装wxPython：

下载 wxpython: http://downloads.sourceforge.net/wxpython/wxPython-src-2.8.12.1.tar.bz2
解压，然后在它的wxPython目录里运行：

    python setup.py build_ext --inplace --debug WX_CONFIG=/opt/wx/2.8/bin/wx-config

WX_CONFIG=/opt/wx/2.8/bin/wx-config 中的路径为configure时设置的路径

(4)设置环境变量：

    export LD_LIBRARY_PATH=/opt/wx/2.8/lib
    export PYTHONPATH=/usr/local/wxPython-2.8.3.0/wxPython

我将wxPython的源码文件夹copy到了/usr/local/ 下，所以第二项是这样
最后将以上两项添加道home目录下的.bashrc中，以后用起来方便点


##三、第一个wxPython程序

写个图形版本的hello world吧，这个程序基本可以看出wxPython程序框架和wxWidget是一致的，只不过代码比C++要简单的多：

    import wx

    class MyFrame(wx.Frame):
        def __init__(self, parent, title):
            wx.Frame.__init__(self, parent, title=title, size=(400,300))
            self.CreateStatusBar() # A StatusBar in the bottom of the window

            # Setting up the menu.
            filemenu= wx.Menu()

            # wx.ID_ABOUT and wx.ID_EXIT are standard ids provided by wxWidgets.
            menuAbout = filemenu.Append(wx.ID_ABOUT, "&About"," Information about this program")
            menuExit = filemenu.Append(wx.ID_EXIT,"E&xit"," Terminate the program")

            # Creating the menubar.
            menuBar = wx.MenuBar()
            menuBar.Append(filemenu,"&File") # Adding the "filemenu" to the MenuBar
            self.SetMenuBar(menuBar)  # Adding the MenuBar to the Frame content.

            # Set events.
            self.Bind(wx.EVT_MENU, self.OnAbout, menuAbout)
            self.Bind(wx.EVT_MENU, self.OnExit, menuExit)


        def OnAbout(self,e):
            # A message dialog box with an OK button. wx.OK is a standard ID in wxWidgets.
            dlg = wx.MessageDialog( self, "Hello world! ", "wxPython Dialog", wx.OK)
            dlg.ShowModal() # Show it
            dlg.Destroy() # finally destroy it when finished.

        def OnExit(self,e):
            self.Close(True)  # Close the frame.

    app = wx.App(False)
    frame = MyFrame(None, "wxPython hello world")
    frame.Show(True)
    app.MainLoop()
