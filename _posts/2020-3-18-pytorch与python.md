﻿---
layout: post
title:  "经典思想FPN介绍"
data: 星期三, 18. 三月 2020 03:09下午 
categories: 视觉
tags: 经典模型
---
* 本篇博客是对计算机视觉中一个经典思想FPN的介绍


#计算机视觉经典思想--FPN

* 这个思想是CVPR2017年提出的，采用特征金字塔做目标检测，后继的很多模型采用了这一思想。

##思想优点
FPN主要解决的是物体检测中的多尺度问题，通过简单的网络连接改变，在基本不增加原有模型计算量的情况下，大幅度提升了小物体检测的性能

FPN更像是一种思想，用这个思想优化的faster-RCNN

##网络结构
![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200318-152031.png)

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200318-152329.png)

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200318-153438.png)

##anchor
对于anchor原本faster-rcnn会采用多尺度加多比例，但是FPN已经考虑了多尺度了，所以只要考虑多比例就行，每一个RPN的一个点只有三个anchor。

{P2,P3,P4,P5,P6}分别对应的anchor面积为{32x32,64x64,128x128,256x256,512x512}

##ablation实验

1.采用类似fig1种的b结构，但是将其他层的特征通过3x3/1x1卷积进行叠加到最高层作为特征输出，因为不同特征层之间存在着semantic gap，从而效果不好
>
![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200318-154148.png)

2.采用top-down金字塔，但是不尽兴lateral相加操作，虽然分辨率也提高了，但是由于没有和对应层的特征信息，从而位置精度不够

3.只采用FPN中的P2分辨率最高的一层作为特征层，并把所有的anchor放在这一层使用FPN是通过固定的窗口来扫描所有特征层，从而增加了尺度不变形的鲁棒性，单独采用P2，特征图尺寸更大，导致产生更多的anchor(750k)，实验说明，更多的anchor并不能带来更好的效果


##为什么FPN能够很好的处理小目标？

1.FPN可以利用经过top-down模型后的那些上下文信息（高层语义信息）；

2.对于小目标而言，FPN增加了特征映射的分辨率（即在更大的feature map上面进行操作，这样可以获得更多关于小目标的有用信息），如图中所示；

##为什么模型融合会使用add而不是concat
也许是经过了实验add效果好一些，而yolo_v3使用的就是concat
