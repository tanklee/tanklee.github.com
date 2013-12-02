---
layout: post
title : 判断大小端
description : 网络编程中有时需要考虑大小端，大小端主要是指数据内部的字节顺序，Little-Endian就是低位字节排放在内存的低地址端，高位字节排放在内存的高地址端。Big-Endian就是高位字节排放在内存的低地址端，低位字节排放在内存的高地址端(像数据流一样填充)。TCP/IP各层协议将字节序定义为Big-Endian，因此TCP/IP协议中使用的字节序通常称之为网络字节序。
category : 算法与数据结构
tags : [C/C++, 算法与数据结构]
---
{% include JB/setup %}

字节序是和平台相关的，常见的CPU架构的字节序吧：

* Big Endian : PowerPC、IBM、Sun
* Little Endian : x86、DEC

写个小程序来判断计算机的大小端：

    #include<iostream>
    using namespace std;

    bool IsLittleEndian()
    {
        short i = 0x0001;

        return((*(char *)&i)? true : false);
    }

    int IsLittleEndian1()
    {
        union
        {
            short i;
            char ibits[sizeof(short)];
        }a;

        a.i = 0x0001;
      
        return (a.ibits[0]? true : false);
    }

    int main(void)
    {
        if (IsLittleEndian())
        {
            cout<<"Little endian"<<endl;
        }
        else
        {
            cout<<"Big endian"<<endl;
        }

        if (IsLittleEndian1())
        {
            cout<<"Little endian"<<endl;
        }
        else
        {
            cout<<"Big endian"<<endl;
        }

        return 0;
    }


