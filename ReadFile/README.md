# 一.概述

C++读写文件有多种方式，下面对常用的几种进行介绍。

# 二.常用方法

这里主要讨论fstream读写文件的相关内容

> #include <iostream>
>
> ofstream	//文件写操作，内存写入存储设备
>
> ifstream	//文件读操作，存储设备读取到内存中
>
> fstream	//读写操作，对打开的文件可进行读写操作

## 1.打开文件

在fstream类中，成员函数open（）实现打开文件的操作，从而将数据流和文件进行关联，通过ofstream,ifstream,fstream对象进行对文件的读写操作

