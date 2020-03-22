---
layout: post
title:  "经典模型Faster-RCNN介绍"
data: 星期二, 17. 三月 2020 03:48下午 
categories: 视觉
tags: 经典模型
---
* 本篇博客是对计算机视觉中一个经典模型YOLO-V3的介绍

#计算机视觉经典模型--Faster-RCNN


* Faster-RCNN做为一个经典模型，单论模型效果在当年还是很惊艳的，但是速度过慢。本篇博文需要对faster-RCNN有一定的了解，并不会讲解很基础的东西，比如9个anchor是什么

##网络结构
![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200317-155605.png)


####模型根据上面的红框大致可以分为三个部分为会一一介绍

##模型输入与输出
![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200317-155709.png)

####我们这里假设输入的数据为[20,M,N,3]

##模块一：预处理网络
####模块输入  [20,M,N,3]
####模块输出  [20,H,W,512]

Faster R-CNN 是采用 VGG16 的中间卷积层的输出（即全连接之前的部分）

如果是ResNet就是下面这样

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200318-093417.png)

##模块二：RPN网络
#### 模块输入 [20,H,W,512]


####分类分支(判断anchor是否有物体)
使用1*1卷积将特征图通道降到18,变为·[20,H,W,18]
>
18是因为一个点有9个anchor，每个anchor都有有物体/没有物体两个预测，相当于二分类问题，所以9*2=18


**reshape:**变为[2,20xHxWx9] 

**softmax:** 归一化

**reshape:**重新变回[20,18,H,W]

但是我们只需要取出前半部分就行了，对于这个anchor是不是有物体的置信度，而不需要判断这个anchor是不是没有物体。

所以分类分支的输出是[20,H,W,9]

####回归分支
使用1x1卷积将特征图维度降到36
>
36是因为每个点有9个anchor，一个anchor4个点所以36个点。

所以回归分支的输出是[20,H,W,36]

>
其实这里回归的输出只是相对与anchor的偏移量，而我们在这一步需要将这些偏移量附加到anchor上面，使anchor变为proposal。
>
![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200317-233802.png)
>
其中tx,ty,tw,th就是我们的预测值(偏移量)，xa表示anchor box的中心点的x值，而图中的x,y,w,h则是我们要转化的proposal的值，其实就是将我们设定好的anchor和回归求出来的偏移量转化为真实的预测框(proposal)

####合并为Proposal（NMS）

首先我们要注意，这里的NMS是对每张图片单独处理，也就是需要循环batch_size

接着对于一张图，我们需要按照分类分支输出的HxWx9个anchor的置信度排序

然后找出分类分支和回归分支的输出拼接在一起，再按照置信度排序，输出[HxWx9,5]的数据作为NMS的输入 (4个坐标加一个置信度)，设置nms_thresh，进行NMS处理。NMS的具体处理可以看我的NMS篇。


NMS处理出的anchor仍旧很多，我们再取其中置信度最高的K个（比如300个），然后将多个图片的拼接在一起，输出也就是[20,K,5]

>
这里可能有人有疑问，我们的目标框已经都求出来了，模型应该已经结束了，但起始这只是一次粗略的回归，还需要下面更加精细的回归。


##模块三： ROIPooling
####模块输入 
RPN生成的proposals[20,K,5]

VGG16最后一层得到的feature map[20,H,W,512]

####模块输出
[20xK,512,7,7]


>
对于每一张图片，我们拿出对应的特征图[H,W,512]，然后根据对应的K个proposal从H*W的特征图中找对对应的部位进行maxpooling。但是proposal是按照原图的大小，这里的H，W相比原图是有缩小的，所以我们将proposal也按比例缩小。

####过程分析
1.Conv layers使用的是VGG16，feat_stride=32(即表示，经过网络层后图片缩小为原图的1/32),原图800*800,最后一层特征图feature map大小:25x25（H，W）

2.假定原图中有一region proposal，大小为665x665，这样，映射到特征图中的大小：665/32=20.78,即20.78x20.78，如果你看过Caffe的Roi Pooling的C++源码，在计算的时候会进行取整操作，于是，进行所谓的第一次量化，即映射的特征图大小为20*20

3.假定pooled_w=7,pooled_h=7,即pooling后固定成7x7大小的特征图，所以，将上面在 feature map上映射的20x20的 region  proposal划分成49个同等大小的小区域，每个小区域的大小20/7=2.86,即2.86x2.86，此时，进行第二次量化，故小区域大小变成2x2

4.每个2x2的小区域里，取出其中最大的像素值，作为这一个区域的‘代表’，这样，49个小区域就输出49个像素值，组成7x7大小的feature map

##模块四：分类与回归网络
![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200317-162051.png)

一共分80类，上面为回归，下面为分类

注意上面的batch_size实际上应该是batch_size x k，k同上面的K，也就是置信度最高的K个框，其实想想也知道一个图片不能只输出一个框。 

####模块输入  [20xk,512,7,7]

1.将输入flatten为[20xk,512x7x7]

2.使用VGG16最后的classifier模块（全连接）[20xk,512x7x7]->[20xk,4096]

3.回归分支为全连接，输出[20xk,4096]->[20xk,4x(class_num+1)] ,coco数据集中class_num为80, 加1是是否为背景的判断。

4.分类分支为全连接，输出[20xk,4096]->[20xk,class_num+1]


## 损失值计算

**注意在RPN的输出和最终输出结果两个地方都需要计算损失值**

损失值有两个部分，回归损失和分类损失

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200318-093803.png)

#### 找个每个ground truth的对应的proposal

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200318-101123.png)

negative label（负标签）：与所有GT包围盒的IoU都小于0.3的anchor。

对于既不是正标签也不是负标签的anchor，以及跨越图像边界的anchor我们给予舍弃，因为其对训练目标是没有任何作用的。

>
在RPN层我们是找出GT是对应哪个anchor（一个anchor计算propsal），但是使用proprosal计算损失。
>
在最后的输出层则是找出GT对应proposal（就是RPN层的proposal），因为已经没有anchor的概念了，然后根据最后的输出结果计算损失

####分类损失
有一点我们需要注意，我们在RPN层只输出了是否有物体，而在最终输出层不仅输出了是否有物体，而且还输出了有什么物体。

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200318-094042.png)

或者是交叉熵损失函数

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200318-094845.png)

>
被丢弃的样本样本不会计算分类损失

####回归损失

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200318-093633.png)

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200318-093641.png)

>
正样本才计算回归损失,负样本和被丢弃的样本都不会计算回归损失

##R-CNN、Fast-RCNN与Faster-RCNN

#### Selective Search（SS）

使用过分割方法将图像分成小区域。在此之后，观察现有的区域。之后以最高概率合并这两个区域。重复此步骤，直到所有图像合并为一个区域位置。

####RCNN解决的是，“为什么不用CNN做classification呢？”

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200318-111728.png)

基于启发式的区域提取方法，用的方法是ss，查看现有的小区域，合并两个最有可能的区域，重复此步骤，直到图像合并为一个区域，最后输出候选区域。然后将根据建议提取的目标图像标准化，作为CNN的标准输入可以看作窗口通过滑动获得潜在的目标图像，在RCNN中一般Candidate选项为1k~2k个即可


问题一：候选框太多

问题二：候选框重复太多

问题三：输入需要固定格式，而归一化过程中对图片产生的形变会导致图片大小改变，这对CNN的特征提取有致命的坏处

** 前两个都是SS的缺点，fast RCNN还有这个缺点

####Fast R-CNN解决的是，“为什么不一起输出bounding box和label呢？”

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20200318-111423.png)

Fast RCNN与RCNN不同。其不同之处如下：Fast RCNN在数据的输入上并不对其有什么限制，而实现这一没有限制的关键所在正是ROI Pooling层。该层的作用是可以在任何大小的特征映射上为每个输入ROI区域提取固定的维度特征表示，解决了第三个缺点

将SS替换掉就是FasterRCNN了

####Faster R-CNN解决的是，“为什么还要用selective search呢？”













