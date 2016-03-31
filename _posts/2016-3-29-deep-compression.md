---
layout: post
title: 阅读笔记 Deep Compression
---
论文全名 [Deep Compression: Compressing Deep Neural Networks with Pruning, Trained Quantization and Huffman Coding](http://arxiv.org/abs/1510.00149)

## 摘要
由于深度神经网络属于计算和内存密集型的算法，因此很难部署在硬件资源受限的嵌入式系统上。本文提出一种叫做"deep Compression"的算法，在不影响精度的情况下网络模型存储空间减小35x至49x。算法的基本流程是模型剪枝，量化训练及霍夫曼编码。模型剪枝，仅学习重要的连接；量化训练，权值量化并强制权值共享；最后使用霍夫曼编码。模型剪枝减少连接数量9x至13x。权值量化使得连接的表达由32位变为5位。在ImageNet数据集上，本文方法减少AlexNet的存储空间为35x，在不损失精度的情况下，从240MB减小为6.9MB。因此模型可以被放到Cache中，而不是内存中，这样不仅提高了速度，也减少了计算的耗电量。
搜索了一下Cache的大小，比如i7-64位寻址; L1 Cache 32KB (注意是Kilo-Byte, 也就是32*8 K-bit); L2 Cache是128KB; L3Cache是2MB per Core (每个核心2MB)。

## Pruning (模型剪枝)
该部分主要基于作者之前工作实现 [Learning both Weights and Connections for Efficient Neural Networks](http://arxiv.org/abs/1506.02626)

Next you can update your site name, avatar and other options using the _config.yml file in the root of your repository (shown below).

![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.
