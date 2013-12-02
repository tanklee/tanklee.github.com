---
layout: post
title: "Boost学习 - 智能指针"
description: "在用C/C++日常编程中，我们经常面临的一个问题，就是要分配一些资源（比内存，网络连接等），然后我们必须负责这些资源的生存期管理，因为这些资源是属于动态资源必须程序员自己负责分配和释放的，要不能可能造成内存泄露。 C/C++中指针和所指向的动态内存，我们使用的时候必须慎之又慎就是因为这个原因，首先我们得确认指针所指向的动态资源是否分配，然后才能使用，使用完后，还得记住去释放。更麻烦的是，如果程序中有多个指针指向同一个动态对象，我们还得确认所有的指针使用完毕才是真正的释放动态对象的时机。总之呢，必须密切关注当前这个对象出于何种状态，有多少个指针在指向它。"
category: Boost
tags: [Boost, C++]
---
{% include JB/setup %}

好吧，小心驶得万年船，我严格考虑到所有引用到资源的指针以及程序所有控制路径，然后做到在合适的时候才去delete并且delete和new总是成对出现的，这会不会有问题了吧。哈哈，还真没这么简单，当异常发生了，程序并没有按照你考虑的正常控制路径执行可能跳过了delete语句，看来这些动态资源，程序员自己手动去管理，真是个吃力不讨好的事。正是基于以上原因，智能指针华丽出现了。


##一、智能指针的作用

那就说说智能指针的作用吧，其实上面已经说的差不多了，归纳一下主要就以下几点
* 避免常见的内存泄露错误，动态对象生存期的自动管理&nbsp;
* 更好地处理多个指针引用同一个动态对象的情形，即共享所有权问题&nbsp;
* 异常发生时，也能安全释放资源，编写异常安全代码&nbsp;

其中的前两点不难理解，异常安全是怎么个情况呢？


##二、异常安全

所谓异常安全就是指异常抛出的时候不会有内存泄露，并且保存程序状态的一致性。几个例子吧，一看你就会明白

    #include <iostream>
    using namespace std;

    class Foo
    {
    public:
        Foo()
        {
            cout<<"Foo::Foo()"<<endl;
        }
        ~Foo()
        {
            cout<<"Foo::~Foo()"<<endl;
        }
        void DoSomething()
        {
            cout<<"Foo::DoSomething()"<<endl;
        }
    };

    void FunMayException(bool bThrow)
    {
        Foo* p = new Foo();
        if (bThrow)
        {
            throw "haha an exception occur";
        }
        p->DoSomething();
        delete(p);
    }

    int main()
    {
        try
        {
            FunMayException(true);
        }
        catch(...)
        {
            cout<<"catch an exception"<<endl;
        }
    }



FunMayException(false)调用，没什么问题，分配个Foo对象，调用个做事函数。FunMayException（true）可出事了，出大事了。函数执行到if语句抛出异常，退出函数，DoSomething函数，delete语句直接跳过，函数栈收缩，释放栈空间，回到main函数处理异常。有delete语句，那个Foo对象却没被析构，内存活生生被泄露了，这个函数是非异常安全的函数。

    [tanli@p04bc cpp]$./test
    Foo::Foo()
    catch an exception

话说我在异常处理里面，去free不就得了， 可是这有多麻烦呢，你得考虑所有异常大的发生的情形，写一大段处理内存泄露的逻辑，这个大的工程里面很不现实啊，还是得智能指针，不管是你自己写个还是用boost库里面的。

##三、STL智能指针auto_ptr

STL中就有个智能指针模板类auto_ptr来自动管理动态资源，说白了，就是RAII，即当我们分配资源时，把资源指针作为参数给一个代理对象初始化，然后这个代理对象就关联了这个资源，我们通过这个代理对象作为代理来访问资源，这个代理对象去完成动态资源的管理，程序员就不去关系释放资源了。所有智能指针就是基于这个思想，下面看看上面的例子，用auto_ptr怎么来避免内存泄露

    void FunMayException(bool bThrow)
    {
        std::auto_ptr<Foo> p(new Foo());
        if (bThrow)
        {
            throw "haha an exception occur";
        }
        p->DoSomething();
    }

看看auto_ptr的用法，和指针差不多。

    void FunMayException(bool bThrow)
    {
        Foo* p = new Foo();
        cout<<"p address:"<<p<<endl;

        std::auto_ptr<Foo> p1(p);
        cout<<"p1 address:"<<p1.get()<<endl;

        cout<<"after copy:"<<endl;
        std::auto_ptr<Foo> p2(p1);
        cout<<"p1 address:"<<p1.get()<<endl;
        cout<<"p2 address:"<<p2.get()<<endl;
       
        if (bThrow)
        {
            throw "haha an exception occur";
        }
        p2->DoSomething();
    }

运行结果：

    [tanli@p04bc cpp]$./test
    Foo::Foo()
    p address:0x502010
    p1 address:0x502010
    after copy:
    p1 address:0
    p2 address:0x502010
    Foo::~Foo()
    catch an exception

auto_ptr的特点：

* 用法和指针一样，可以`*`和`->`操作符访问绑资源。只是auto_ptr对象析构函数会释放所绑定资源，即离开作用域会自动释放资源
* auto_ptr不具备共享所有权语义，即对auto_ptr对象拷贝或赋值后，原auto_ptr失去所有权（内部指针值为NULL）
* 不能用于动态对象数组，不能用于STL容器元素

看到auto_ptr的局限，我们可以开始了解boost里面的智能指针了。


##四、boost智能指针scoped_ptr
boost::scoped_ptr和std::auto_ptr几乎是一样的，就是用一个智能指针对象作为代理来管理动态对象，智能指针类重载了* ->运算符，并且会在析构时释放我们的释放我们申请的动态对象。

    void FunMayException(bool bThrow)
    {
        boost::scoped_ptr<Foo> p(new Foo());
        if (bThrow)
        {
            throw "haha an exception occur";
        }
        p->DoSomething();
    }

唯一的差别就是scoped_ptr是对资源的所有权是独有的，不允许进行复制和赋值。上面我们说到auto_ptr的时候说到auto_ptr复制或赋值的时候回转移所有权，原来的auto_ptr对象会保存一个空指针。这就有问题了，如果用auto_ptr完成自动管理，但其间不小心赋值转移了所有权，那么再用这个auto_ptr去引用资源，就会出现未定义的行为。scoped_ptr做为管理一个小范围动态对象的智能指针，则是直接拒绝这种转移所有权的行为，即scoped_ptr 是个不能复制和赋值的类，直接报编译错误，从而让避免用户犯错。

    void FunMayException(bool bThrow)
    {
        Foo* p = new Foo();
        cout<<"p address:"<<p<<endl;

        boost::scoped_ptr<Foo> p1(p);
        cout<<"p1 address:"<<p1.get()<<endl;

        cout<<"after copy:"<<endl;
        boost::scoped_ptr<Foo> p2(p1);
        cout<<"p1 address:"<<p1.get()<<endl;
        cout<<"p2 address:"<<p2.get()<<endl;
       
        if (bThrow)
        {
            throw "haha an exception occur";
        }
        p2->DoSomething();
    }

编译下出现错误：

    /usr/local/brion/boost/1.37.0/include/boost/scoped_ptr.hpp:45: error: `boost::scoped_ptr<T>::scoped_ptr(const boost::scoped_ptr<T>&) [with T = Foo]' is private
    test.cpp:31: error: within this context

因此如果要管理一个被限制了作用域的资源，而且还要是资源的唯一拥有者，即不能转移所有权，那么就使用boost：；scoped_ptr吧

##五、boost智能指针shared_ptr

对于std::auto_ptr和boost::scoped_ptr,分配的动态资源只能绑定到一个智能指针对象，这个智能指针独占所有权负责资源的生存期，而实际的情形时我们有时候需要在两个或多个智能指针对象共享资源，那么shared_ptr就浓重登场了。

    #include "boost/shared_ptr.hpp"
    #include <iostream>

    using namespace std;

    class Foo
    {
    public:
        Foo()
        {
            cout<<"Foo::Foo()"<<endl;
        }
        ~Foo()
        {
            cout<<"Foo::~Foo()"<<endl;
        }
        int m_no;
    };

    class A
    {
    public:
        boost::shared_ptr<Foo> m_pFoo;
       
        A(boost::shared_ptr<Foo> pFoo)
        {
            m_pFoo = pFoo;
        }
    };

    class B
    {
    public:
        boost::shared_ptr<Foo> m_pFoo;
       
        B(boost::shared_ptr<Foo> pFoo)
        {
            m_pFoo = pFoo;
        }
    };

    int main()
    {
        boost::shared_ptr<Foo> temp(new Foo());
        temp->m_no = 100;

        A a(temp);
        cout<<"a.m_pFoo->m_no:"<<a.m_pFoo->m_no<<endl;
        cout<<"count:"<<temp.use_count()<<endl;

        {
            B b(temp);
            cout<<"b.m_pFoo->m_no:"<<b.m_pFoo->m_no<<endl;
            cout<<"count:"<<temp.use_count()<<endl;
        }
       
        cout<<"B out of scoped"<<endl;
        cout<<"count:"<<temp.use_count()<<endl;
    }

运行一下：

    [tanli@p04bc boost]$./test
    Foo::Foo()
    a.m_pFoo->m_no:100
    count:2
    b.m_pFoo->m_no:100
    count:3
    B out of scoped
    count:2
    Foo::~Foo()

shared_ptr通过引用计数来管理资源的生存期，多个智能指针对象共享对资源的所有权，直到最后一个智能指针对象析构时才会释放资源。这样我们当我们需要共享资源时，多个指针之间就不用编写复杂的管理共享资源的逻辑。因为boost：：shared_ptr具备复制和赋值的语义。正因为如此，share_ptr可以放入stl容器，比如我们要存一些对象到容器，如果直接存对象的话，首先复制开销大，对于多态对象会被切割，失去多态性；而存裸指针到容器，就要管理生存期，相当复杂，shared_ptr就没有这些问题了。总结一下：


* 对个智能指针对象共享资源的所有权&nbsp;
* 可以放心复制和赋值不必担心生存期&nbsp;
* 可以放入stl容器


##六、boost智能指针weak_ptr

有时候我们仅仅需要观察资源的生存期，但却不需要资源的所有权。这时候boost::weak_ptr就出现了，weak_ptr通常和shared_ptr一起使用，扮演者shared_ptr观察者的角色。当一个被weak_ptr观察的shared_ptr释放其绑定的资源时候， 观察者weak_ptr会得到通知，更新weak_ptr指针为空，这样可以防止当资源释放以后悬空的指针。因为weak_ptr不具有所有权，所有不会影响引用计数。

    #include "boost/shared_ptr.hpp"
    #include "boost/weak_ptr.hpp"
    #include <iostream>
    #include <cassert>

    using namespace std;

    class Foo
    {
    public:
        Foo()
        {
            cout<<"Foo::Foo()"<<endl;
        }
        ~Foo()
        {
            cout<<"Foo::~Foo()"<<endl;
        }
    };

    int main()
    {
        boost::weak_ptr<Foo> wp;
       
        assert(wp.expired()); // Don't bind any source, or the source have been release
        {
            boost::shared_ptr<Foo> sp1(new Foo());
            assert(sp1.use_count() == 1);

            wp = sp1; // Observe the source which shared_ptr sp1 bind
            assert(sp1.use_count() == 1);        // Don't add the source's count
            assert(wp.use_count() == 1);         // weak_ptr can also get the source's count

            boost::shared_ptr<Foo> sp2(wp);      // Create a shared_ptr from the weak_ptr
            assert(sp1.use_count() == 2);
            //assert(sp2 == wp);
        }

        assert(wp.expired());
        boost::shared_ptr<Foo> sp3 = wp.lock();  // Get a shared_ptr from weak_ptr
        assert(sp3 == NULL);
    }

weak_ptr做为资源的观察者，并不具备所有权，所有不能直接访问资源，这也是为什么weak_ptr并不重载* -》运算符的原因，因为访问的是不安全的，它没有引用计数，资源生存期无法控制。但是最终我们还是想访问的，就还得要与shared_ptr 合作了，把weak_ptr转换成shared_ptr就可以访问了，转换的方有两种

	* weak_ptr传给shared_ptr构造函数shared_ptr()(const weak_ptr<T>& r)
	* weak_ptr成员函数 shared_ptr<T> lock()

区别在于是否把空的weak_ptr当做一种错误，第一种方式，构造函数会抛出一个bad_weak_ptr类型异常，第二种会返回一个空的shared_ptr

    int main()
    {
        boost::shared_ptr<string> sp(new std::string("Some resource"));
        boost::weak_ptr<string> wp(sp); //wp is observer of sp

        if (boost::shared_ptr<string> sp1 =  wp.lock()) // from wp.lock to get shared_ptr
        {
            cout<<"Got it:"<<*sp1<<endl;
        }
        else
        {
            std::cout<<"Shared_ptr is empty"<<endl;
        }

        sp.reset; //sp release ownership, it is NULL now

        try
        {
            boost::shared_ptr<string> sp2(wp); //from wp create shared_ptr
        }
        catch(boost::bad_weak_ptr e)
        {
            cout<<"exception:"<<e.what()<<endl;
        }
    }

总结一下对于weak_ptr 适用场合

* 观察资源状态，并不需要资源所有权&nbsp;
* 可以用于避免悬空指针，因为资源释放时，裸指针还是保存一个回收的地址值，而 weak_ptr作为观察者会更新为空
* 可用于打破循环依赖&nbsp;


