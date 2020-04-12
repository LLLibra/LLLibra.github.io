---
layout: post
title:  "STL - vector"
data: 星期四, 12. 三月 2020 03:42下午 
categories: 数据结构
tags: 专题
---
* 该模块会针对数据结构中的某一块知识做专题整理，也许会有些不足或者错误的地方，未来可能会作修改。

#  数据结构专题1----STL - vector

## 底层数据结构
动态数组

## vector 特点
1.拥有一段连续的内存空间，并且起始地址不变，因此能够非常好的支持随机存取，即[]操作符

2.因为内存是连续的，所以你原来申请的空间可能后面就无法扩容了，需要重新申请一块足够大得内存并且进行内存的拷贝，这些都大大的影响了vector的效率。（这也就解释了，为什么 vector 容器在进行扩容后，与其相关的指针、引用以及迭代器可能会失效的原因。）

3.对头部和中间进行添加删除元素操作需要移动内存，如果你得元素是结构或类，那么移动的同时还会进行构造和析构操作（所以如果元素是结构或类，最好是将结构或类的指针放入vector中，这样不仅能够节省空间，而且可以避免移动时构造和析构操作），所以性能不高。删除元素时采用后面的元素覆盖前面的元素的方法可以提高效率




## vector常用函数


#### 查看属性
a.front() //返回当前vector容器中起始元素的引用。

a.begin() //返回一个当前vector容器中起始元素的迭代器。

a.back()

a.size()

a.empty()

a.capacity() //判断vector的容量
#### 插入
a.push_back（）
>
C++11中，针对顺序容器(如vector、deque、list)，新标准引入了三个新成员：emplace_front、emplace和emplace_back，这些操作构造而不是拷贝元素。这些操作分别对应push_front、insert和push_back，允许我们将元素放置在容器头部、一个指定位置之前或容器尾部。
>
当调用push或insert成员函数时，我们将元素类型的对象传递给它们，这些对象被拷贝到容器中。而当我们调用一个emplace成员函数时，则是将参数传递给元素类型的构造函数。emplace成员使用这些参数在容器管理的内存空间中直接构造元素。
>
而原本的情况，如果是右值，我们可能还要指向构造函数+拷贝函数，起码也要执行拷贝函数，但是用emplace我们就可以直接是使用构造函数。该元件是构成在就地，即没有复制或移动操作进行。

a.insert（）

>
//在a的第一个元素（从第0个算起）位置插入数值5,
>
a.insert(a.begin()+1,5);
>
//在a的第一个元素（从第0个算起）位置插入3个数，其值都为5
>
a.insert(a.begin()+1,3,5);
>
//b为数组，在a的第一个元素（从第0个元素算起）的位置插入b的第三个元素到第5个元素（不包括b+6）
>
a.insert(a.begin()+1,b+3,b+6);

#### 删除
a.clear()
>
clear时，清空了对应内存中的所有数据，size的结果为0，但是内存并没有被释放，若此时调用capacity，并不会改变容量大小，若想释放，需用swap将原来的数据复制与新的内存中，则原内存会被释放.
>
   注意：并不是所有的STL容器的clear成员函数的行为都和vector一样。事实上，其他容器的clear成员函数都会释放其内存。比如另一个和vector类似的顺序容器deque。

a.erase()
>
iterator erase(iterator position);
>
iterator erase(iterator first, iterator last);
>
不包括 last
>
在进行单个元素删除后，传入的迭代器指向不变，仍然指向被删除元素的位置，而被删除元素之后的所有元素都向前移动一位，也就是该迭代器实际上是指向了原来被删除元素的下一个元素。

pop_back();返回尾元素

> #### remove
vector中的remove的作用是将等于value的元素放到vector的尾部，但并不减少vector的size

#### 初始化
**方法一：**
>
创建一个容量为10的空vector
>
vector< int >a(10);

**方法二：**
>
//定义具有10个整型元素的向量，且给出的每个元素初值为1
>
vector< int >a(10,1);

**方法三：**
>
**b也为vector，a的值完全等于b的值**
>
vector< int >a(b);
>
**将向量b中从0-2（共三个）的元素赋值给a，a的类型为int型**
>
vector< int >a(b.begin(),b.begin+3);


**方法四：**
>
int b[7]={1,2,3,4,5,6,7};
>
vector< int > a(b,b+7）;



#### 其他
a.resize(100)
>
只有当resize的大小大于原本申请的内存时才会去另外申请空间。而resize改变了vector的capacity同时也增加了它的size！

a.reserve(100)
>
vector 的reserve增加了vector的capacity，但是它的size没有改变！



a.swap()
>
a.swap(b)
>
b为另外一个vector
>
使用swap释放内存
>
![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200313-181318.png)


==判断相等

## vector原理简单分析（只知道地层是动态数组，其他的暂不确定）
主要是依靠三个指针

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200313-144651.png)

size表示vector中已有元素的个数，容量表示vector最多可存储的元素的个数。

对于空的 vector 容器，由于没有任何元素的空间分配，因此 _Myfirst、_Mylast 和 _Myend 均为 null。



## vector的扩容
为了降低二次分配时的成本，vector实际配置的大小可能比客户需求的更大一些，以备将来扩充，这就是容量的概念。即capacity>=size，当等于时，容器此时已满，若再要加入新的元素时，就要重新进行内存分配，整个vector的数据都要移动到新内存。二次分配成本较高，在实际操作时，应尽量预留一定空间，避免二次分配。

当通过push_back向其中增加元素时，如果初始分配空间已满，就会引起vector扩容，其扩容规则在gcc下直接将当前容量翻倍。

vector优异性能的秘诀之一，就是配置比其所容纳的元素所需更多的内存，一般在使用vector之前，就先预留足够空间，以避免二次分配，这样可以使vector的性能达到最佳。因此元素个数_Count是个远比元素值 _Val重要的参数，因此当构造一个vector时，首要参数一定是元素个数。

## vector的析构
vector的析构函数很简单，就是先销毁所有已存在的元素，然后释放所有内存

vector的clear会清除所有内容，size会为0,但是capacity仍旧保持不变，内存并没有释放

正确的释放方法：pointVec.swap(vector< Point > ())

## 如何避免扩容
就是需要事先分配足够的空间，然后可以 调用shrink_to_fit()，capacity会缩小到合适大小

或者vector< int >(ivec).swap(ivec);

## vector和数组的效率

数组 > 动态数组 > 预先reverse的vector > vector


## vector删除元素
注意两点

1.erase(iter)会自动将iter指向你要删除的元素的下一个元素的位置。

2.最好写成iter=vt.erase(iter)


## C++中为什么要用vector代替数组
1.编译器和运行时系统都不会检查数组下标是否位于正确的范围之内；

2.数组的大小可以是固定不变的，或者必须使用堆中的动态内存；

（当数组大小固定的时候还是可以考虑数组）


## 番外 Array
Array相当于vector和数组的结合体，但是和vector更像一些。

位于std命名空间 #include< array >

![](imgs/20200328-204222.png)

1.vector属于变长容器，即可以根据数据的插入删除重新构建容器容量；但array和数组属于定长容量。

2.vector提供了可以动态插入和删除元素的机制，而array和数组则无法做到，或者说array和数组需要完成该功能则需要自己实现完成

相对于数组来说 array是一个容器，虽然有很多不能用，但是还是优于数组（可以使用比如拷贝和赋值、swap函数）


array的空间在栈上


## 残留问题：
为什么vector的效率低于数组，即使预分配了空间。

如果删掉了vector尾部的元素 那么尾部的迭代器属于什么

iter=vt.erase(iter)和 vt.erase(iter) 什么区别
