---
layout: post
title:  "GDB调试"
data: 星期四, 26. 三月 2020 09:51下午 
categories: linux
tags: 专题
---
* 该模块会针对linux中的某一块知识做专题整理，也许会有些不足或者错误的地方，未来可能会作修改。

#linux专题2----GDB调试


g++ -g test.c -o test（-g选项告诉gcc在编译程序时加入调试信息）

gdb test

然后你就会看到出现好多信息在屏幕上，大致说的是gdb的一些版本信息说明之类的，但是它对你调试程序没用呀，所以，你可以加上-q选项，不输出它们。

**gdb -q test**

也可以先进gdb再加载文件 

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200331-174117.png)


**shell clear** 
清屏


**break 6**

在第6行设置断点

**info breakpoints**

查看断点信息

#### 查看类的内存分布

set p pretty on //可以使得打印信息按层次分布 更直观




