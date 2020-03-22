﻿---
layout: post
title:  "Effective C++ (2)"
data: 星期五, 14. 二月 2020 10:03上午 
categories: C++
tags: 读书笔记
---
* 该模块会针对我读一本书进行整理，会分几篇博客完成，也许会有些不足或者错误的地方，未来可能会作修改。

* 有一些是看了其他博客觉得比较好摘来的，有一些是书上的，有一些是自己总结的。

# Effective C++总结2

## 条款11：在 operator= 中处理“自我赋值”
由于变量有别名的存在（多个指针或引用只想一个对象），所以可能出现自我赋值的情况。但是自我赋值的过程是先删除自身，如果自我赋值就会出问题。

1.先进行对象是否相同的检查。

2.先创建一个temp对象指向本对象，然后令本对象复制目标对象，然后删除temp对象（原本对象）。

3.先创建一个temp对象复制目标对象，然后交换temp对象与本对象。

## 条款12：复制对象时勿忘其每一个成分
复制构造函数和赋值构造函数要确保复制了对象内的所有成员变量和所有基类成分，这意味着你如果自定义以上构造函数，那么每增加成员变量，都要同步修改以上构造函数，且要调用基类的相应构造函数。

复制构造函数和赋值构造函数看似代码类似，但不要用一个调用另一个，好的做法是建立一个private的成员函数来做这件事，然后两个构造函数都调用该成员函数。

## 条款13：以对象管理资源
主要就是使用智能指针，可以看我的智能指针篇。

## 条款14：在资源管理类中小心copying行为
不是所有资源都适合用智能指针管理，所以需要建立自己的资源管理类。

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200214-101903.png)

## 条款15：在资源管理类中提供对原始资源的访问
封装了资源管理类后，API有时候往往会要求直接使用其原始资源，一般有两种方法。

** 显式转换方法： **如指针的->和(*)操作，也比如自制一个getXXX()函数

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200214-112230.png)

** 隐式转换方法： **比如覆写XXX()取值函数

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200214-124506.png)

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200214-124531.png)

显式操作比较安全，隐式操作比较方便（但容易被误用）。

## 条款16：成对使用new和delete时要采取相同形式
使用new申请内存，必须使用delete释放内存，使用new [] 申请内存，必须使用delete []释放内存。

对于数组，不建议使用typedef行为，这会让使用者不记得去delete []。对于这种情况，建议使用string或者vector。

## 条款17：以独立语句将newed对象置入智能指针
即注意在使用资源前要将资源放入资源管理器中（比如智能指针）。

## 条款18：让接口容易被正确使用，不易被误用
好的接口要容易被正确使用，不容易被误用，符合客户的直觉。

1.促进正确使用的办法包括保持接口的一致性，既包括自定义接口之间的一致性，也包括与内置类型行为的相似一致性。

2.阻止误用的办法包括建立新类型来限制该类型上的操作、束缚对象的值以及消除客户管理资源的责任，以此来作为接口的参数与返回类型。

3.shared_ptr支持定制删除函数，所以可以很方便的实现上述问题，以及防范DLL问题。

## 条款19：设计class犹如设计type
在设计class时，要考虑一系列的问题，包括

1.对象的创建和销毁（构造、析构）

2.对象的初始化与赋值（构造、赋值操作符）

3.复制操作（复制构造）

4.合法值（约束条件）

5.继承体系（注意虚函数）

6.支持的类型转换（显示转换、类型转换操作符）

7.成员函数和成员变量的可见范围（public/protected/private）

8.是否用模板就能实现？

## 条款20：宁以传递const引用替换传递值
1.除非你另外指定，否则函数参数都是以实际实参的副本为初值，返回的也是返回值的一个副本。这些副本都是由对象的拷贝构造函数产出的。但其实这是十分浪费时间的事，虽然十分有效。最有效的是传入引用，最好是常量引用，不然原本的实参有被改变的风险。

2.一般来说只有在传 内置类型和STL的迭代器和函数对象时才会认为 传入值时更高效的，其它的情况还是传入常量引用比较好。

3.传入值的还有一个问题就是** 对象切割问题 **，简单说就是当函数的形参是父类，你传入的是值，同时你要调用一个虚函数，那么这个函数就是父类的虚函数，但是如果你传入的是引用，那么这个函数就是子类的函数。指针同引用。

## 条款21：必须返回对象时，别妄想返回其引用
一个局部变量在离开函数之前就被销毁了，所以不可能返回局部变量的引用。

一个必须返回新对象的函数就应该让那个函数返回一个新对象而不是引用。

## 条款22：将成员变量声明为private
切记将成员变量声明为private，这可以保证客户访问数据的一致性、可以细微划分访问控制、允许约束条件获得保证，并提供类作者充分的实现弹性来修改对其的处理，因为这保证了“封装性”，作者可以改变实现和对成员变量的操作，而不改变客户的调用方式。

protected并不比public更加具有封装性，有一个public成员变量，如果取消此变量，则所有使用它的客户码都会被破坏，而那是一个不可知的大量。因此public成员变量完全没有封装性。假设一个protected成员变量，最终取消此变量，则所有使用它的derived classes都会被破坏，而那往往也是一个不可知的大量。因此，protected成员变量就像public成员变量一样缺乏封装性。

## 条款23：宁以非成员、非友元替换成员函数
文中提到了封装性强弱的概念：一个类中的成员或者成员函数，被越少的代码访问，那么封装性越高。所以增加一个成员函数会增加访问成员变量的代码，因此降低了封装性。

宁可拿非成员非友元函数来替换成员函数。因为这种函数位于函数之外，不能访问类的private成员变量和函数，保证了封装性（没有增加可以看到内部数据的函数量）。

此外，这些函数只要位于同一个命名空间内，就可以被拆分为多个不同的头文件，客户可以按需引入头文件来获得这些函数，而类是无法拆分的（子类继承与此需求不同），因此这种做法有更好的扩充性。

## 条款24：若所有参数皆需类型转换，请为此采用非成员函数

如果你要为某个函数的所有参数（包括this所指对象本身）进行类型转换，那么该函数必须是个非成员函数。

比如说axb，a中有operator * 的函数，这个运算可以成立，将b转换为和a一个类型进行运算。但是b中没有，所以bxa就会出错，要想bxa不出错，就要将operator*写成一般函数（非成员函数），将a和b都进行转换。


一个与该class有关的函数可以只是普通函数 而不一定必须是成员函数或者friend函数。

## 条款25：考虑写出一个不抛异常的swap函数
由于swap函数如此重要，需要特别对他做出一些优化。

常规的swap是简单全复制三次对象进行交换（包括temp对象），如果效率足够就用常规版。

如果效率不够，那么给你的类提供一个成员函数swap，用来对那些复制效率低的成员变量（通常是指针）做交换。然后，提供一个非成员函数的swap来调用这个成员函数，供别人调用置换。

对于类（非模板），为标准std::swap提供一个特定版本（swap是模板函数，可以特化）。

在类内的swap交换内置类型时要调用std命名空间内的swap函数，必须使用using std::swap，否则就变成递归函数了。编译器会自行从所有可能性中选择最优版本。

## 条款26：尽可能延后变量定义式的出现时间

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200214-110847.png)

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200214-110852.png)

## 条款27：尽量少做转型操作

转型返回的不是引用而是一个副本。这是转型容易出错的一个原因

尽量避免使用转型cast（包括C的类型转换和C++的四个新式转换函数），特别是注重效率的代码中避免用dynamic_casts。

如果一定要用，试着考虑无需转型的替代设计，例如为基类添加一个什么也不做的衍生类使用的函数，避免在使用时需要将基类指针转型为子类指针。

如果一定要转型，试着将其隐藏于某个函数后，客户调用该函数而无需自己用转型。

宁可使用C++新式转型，也不用用C的旧式，因为新式的更容易被注意到，而且各自用途专一。


![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200214-111216.png)

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200214-111334.png)

* 想要了解更多细节可以看我整理的关于4种转型的文章

## 条款28：避免返回handles指向对象内部成分
避免让外部可见的成员函数返回handles（包括引用、指针、迭代器）指向对象内部

一个类的封装性最多只等于返回其引用的函数访问级别。

简单来说就一个类中a属性本来是private的，但是有个函数返回了它的引用，那么外部就可以修改a，a就是不private。但其实返回的是指针和迭代器都有同样的问题。

虽然将函数的返回值改为常量引用就可解决这个问题，但是由于返回的还是引用，用户是可能将这一变量删除的。比如在一个函数里拿到了这个引用，离开了这个函数引用就没了。

## 条款29：为异常安全而努力是值得的
　发生异常时的处理主要分一下几类：资源不泄漏、数据不丢失、不抛出异常。反正就是考虑程序的各种可能的情况，如果异常了要尽可能保证你的程序某些功能或数据不丢失。这个也没啥可说的。


## 条款30：透彻了解inline的里里外外
有一篇专门的博客会讲这些