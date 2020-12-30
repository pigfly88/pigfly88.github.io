---
layout: post
category: "php"
title:  "java简介"
---

Java介于编译型语言和解释型语言之间。编译型语言如C、C++，代码是直接编译成机器码执行，但是不同的平台（x86、ARM等）CPU的指令集不同，因此，需要编译出每一种平台的对应机器码。解释型语言如PHP、Python、Ruby没有这个问题，可以由解释器直接加载源码然后运行，代价是运行效率太低。

JRE：Java Runtime Environment，包含运行Java字节码的虚拟机JVM。JDK：Java Development Kit，包含JRE，还有把源码编译成字节码的编译器。

计算机内存的最小存储单元是字节(byte)，1 byte = 8 bit，Java用来存放数值的类型有：byte(8), short(16), int(32), long(64), float(32), double(64)。

Java提供了两种变量类型：基本类型和引用类型。基本类型包括：整型、浮点型、布尔型，字符型，其他为引用类型。引用类型的变量类似于C语言的指针，它存储的内存地址，指向某个对象在内存中的位置。

