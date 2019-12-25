---
layout: post
title:  "经典Normalization介绍"
data: 星期四, 19. 十二月 2019 03:57下午 
categories: 视觉
tags: 基础结构
---
* 本篇博客是对计算机视觉模型中经常使用到的结构Nromalization的一个介绍和总结。

# 计算机视觉经典结构--Normalization
### 作用：
 在去年的论文《How Does Batch Normalization Help Optimization?》里边提出了自己关于BN的新理解他们认为BN主要作用是使得整个损失函数的landscape更为平滑，从而使得我们可以更平稳地进行训练。其他的Normal方式的作用也大同小异。


### Batch Normalization（2015）
出现最早的的归一话方式，一般的网络输入为NCHW，N为batch大小，C为通道数，一般为3,H和W为图片的宽和高。BN也是我目前用的最多的归一化。

#### 宏观的公式：

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20191224-211936.png)

#### 微观的公式：

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20191224-212107.png)

网上看到一个比较好比喻，假设我有N本书，每本书C页，每页H行，每行W个数字，均值就是求平均每页的每个数字是多少，方差同理。

**作用：**去年的论文《How Does Batch Normalization Help Optimization?》里边，认为BN主要作用是使得整个损失函数的landscape更为平滑，从而使得我们可以更平稳地进行训练。

**缺点：**难于用在RNN（循环神经网络上），并且当batch小的时候效果不好。

**注意：**在测试的时候不一定batch，有batch也不一定一样，这个时候用的均值和方差是全量训练数据的均值和方差，这个可以通过移动平均法求得。

--------

### Layer Normalization（2016）
主要是针对 batch normalization 存在的问题进行改进，不同于BN，可以在单条数据内部就能归一化。

Layer normalization 对于recurrent neural networks 的帮助最大。

Layer normalization 对于 Convolutional Networks 作用不是很大，后续研究可以提高其作用。

#### 公式：

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20191224-212906.png)

如果按照之前的比喻，这就是求平均每本书的一个数字是多少。

--------

### Instance Normalization（2016）
最初用于图像的风格迁移。作者发现，在生成模型中， feature map 的各个 channel 的均值和方差会影响到最终生成图像的风格，因此可以先把图像在 channel 层面归一化，然后再用目标风格图片对应 channel 的均值和标准差“去归一化”，以期获得目标图片的风格。IN 操作也在单个样本内部进行，不依赖 batch。


#### 公式：

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20191224-213059.png)

只在每本书里面求，求平均每页里一个数字是多少

--------

### Group Normalization（2018）
GN适用于占用显存比较大的任务，例如图像分割。对这类任务，可能 batchsize 只能是个位数，再大显存就不够用了。而当 batchsize 是个位数时，BN 的表现很差，因为没办法通过几个样本的数据量，来近似总体的均值和标准差。

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20191224-213644.png)

把每一个样本 feature map 的 channel 分成 G 组，每组将有 C/G 个 channel，然后将这些 channel 中的元素求均值和标准差。各组 channel 用其对应的归一化参数独立地归一化。

相当于将通道分成多份，每份进行一次Batch Normalization。

这么看GN和 Layer Normalization 和  Instance Normalization起始都可以用于batchsize比较小的时候，但是真实比起来还是GN的效果比较好

#### 公式：

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20191224-213653.png)

--------

### Filter Response Normalization (2019)

#### 公式：

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20191225-131531.png)

跟 正常的 batch normalization 相比，FRN有两个不同：

1. 都是在 channel level 做 normalization 的，但是 FRN 没有在 batch 这个维度上求 variance （其实相当于 batch size=1）。自然不会受制于 batch size
2. FRN 里面，没有减去 mean。

#### Thresholded Linear Unit (TLU)
因为上面缺少了 mean centering 部分，所以我们这里用到一个新的激活函数。这个激活函数说来也简单，就是让 ReLU 是可以 learn 的。

![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20191225-131824.png)

这里的![](https://github.com/LLLibra/LLLibra.github.io/raw/master/_posts/imgs/20191225-131839.png)就是那个可以 learn 的 threshold。

TLU 不会让 正常 BN 变差，但是能让 FRN 大幅度变好；









