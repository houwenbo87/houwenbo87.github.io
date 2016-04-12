---
layout: post
title: 阅读笔记 Deep Compression
---
论文链接 [Deep Compression: Compressing Deep Neural Networks with Pruning, Trained Quantization and Huffman Coding](http://arxiv.org/abs/1510.00149)

## 摘要
深度神经网络属于计算和内存密集型的算法，很难部署在硬件资源受限的嵌入式系统上。

该论文提出一种叫做"deep Compression"的算法，在不影响精度的情况下网络模型存储空间减小35x至49x。

算法的基本流程是模型剪枝、量化训练、霍夫曼编码。模型剪枝，仅学习重要的连接；量化训练，权值量化并强制权值共享；最后使用霍夫曼编码。

模型剪枝减少连接数量9x至13x。权值量化使得连接的表达由32位变为5位。

在ImageNet数据集上，本文方法减少AlexNet的存储空间为35x，在不损失精度的情况下，从240MB减小为6.9MB。这样模型可以被放到Cache中，不仅提高了速度，也减少了计算的耗电量。在网络带宽受限或手机应用文件大小限制等方面，模型尺寸的压缩也有很大的益处。在CPU，GPU及移动端GPU上评测，压缩的网络有3x至4x的加速，并且有3x至7x的耗电减少。

搜索了一下Cache的大小，比如i7-64位寻址; L1 Cache 32KB (注意是Kilo-Byte, 也就是32*8 K-bit); L2 Cache是128KB; L3Cache是2MB per Core (每个核心2MB)。

![_config.yml]({{ site.baseurl }}/images/deep-compression/compression.png)

## Network Pruning (模型剪枝)
该部分主要基于作者之前工作实现，细节详见 [Learning both Weights and Connections for Efficient Neural Networks](http://houwenbo87.github.io/prune-connections/)

## Trained Quantization and Weight Sharing (权值量化及共享)
该部分主要通过减少权值表达的bits数量来进一步压缩模型的存储空间。多个connection共享相同的权值来限制有效权值的数量，然后finetuning这些共享的权值。

权值共享示意图如下图所示，假设某一层有4个输入和4个输出，改成的权值是一个4x4的矩阵。图中左上是权值矩阵，左下是梯度矩阵。权值被量化为4组，每组用相同的颜色表示。因此权值可以用索引和一个共享权值表来表示，减少存储空间。在权值更新的时候，对相同组内与权值对应的梯度值进行累加取平均，然后对权值表中的权值进行更新。对于剪枝后的AlexNet，卷积层量化为8-bits(256个共享权值)，全连接量化为5-bits(32个共享权值)，模型精度没有任何损失。

![_config.yml]({{ site.baseurl }}/images/deep-compression/weight_sharing.png)

在已训练的模型基础上，使用k-means聚类方法对权值进行分组，这样聚类为一组的连接共享相同的权值。注意在层间不进行权值共享。

k-means的初始化方法，对模型的预测精度有一定的影响。采用在权值的[min,max]之间均匀取值作为k-means的初始值。由于权值中趋近于0的数占大多数，而越接近于0的权值重要程度越低，所以不能选择随机初始化或与权值分布相关的初始化方式。

前向和后向过程。k-means的聚类中心为共享的权值，前向和后向计算的时候通过查表来获取权值，权值更新方式如上图。

霍夫曼编码是一种通用的数据无损压缩方式，这里不做具体介绍。

作者YouTube上的讲解 https://www.youtube.com/watch?v=baZOmGSSUAg
