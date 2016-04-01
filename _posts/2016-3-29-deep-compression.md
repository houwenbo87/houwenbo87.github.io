---
layout: post
title: 阅读笔记 Deep Compression
---
论文全名 [Deep Compression: Compressing Deep Neural Networks with Pruning, Trained Quantization and Huffman Coding](http://arxiv.org/abs/1510.00149)

## 摘要
深度神经网络属于计算和内存密集型的算法，很难部署在硬件资源受限的嵌入式系统上。

该论文提出一种叫做"deep Compression"的算法，在不影响精度的情况下网络模型存储空间减小35x至49x。

算法的基本流程是模型剪枝、量化训练、霍夫曼编码。模型剪枝，仅学习重要的连接；量化训练，权值量化并强制权值共享；最后使用霍夫曼编码。

模型剪枝减少连接数量9x至13x。权值量化使得连接的表达由32位变为5位。

在ImageNet数据集上，本文方法减少AlexNet的存储空间为35x，在不损失精度的情况下，从240MB减小为6.9MB。这样模型可以被放到Cache中，不仅提高了速度，也减少了计算的耗电量。在网络带宽受限或手机应用文件大小限制等方面，模型尺寸的压缩也有很大的益处。在CPU，GPU及移动端GPU上评测，压缩的网络有3x至4x的加速，并且有3x至7x的耗电减少。

搜索了一下Cache的大小，比如i7-64位寻址; L1 Cache 32KB (注意是Kilo-Byte, 也就是32*8 K-bit); L2 Cache是128KB; L3Cache是2MB per Core (每个核心2MB)。

## Pruning (模型剪枝)
该部分主要基于作者之前工作实现，细节详见 [Learning both Weights and Connections for Efficient Neural Networks](http://arxiv.org/abs/1506.02626)

Pruning就是去掉网络中的冗余连接。如下图所示，主要有三个步骤：1.学习那些连接是重要的；2.去掉不重要的连接；3.finetuning去掉冗余连接后的网络。
![_config.yml]({{ site.baseurl }}/images/deep-compression/pruning_1.png)

所谓训练连接的重要性，就是正常训练的模型，权值越大表示这个连接越重要，反之就是冗余的连接，可以剪枝掉。所谓模型剪枝就是设定一个阈值，将权值小于该阈值的连接去掉。如下图所示，将稠密网络转换为稀疏网络。
![_config.yml]({{ site.baseurl }}/images/deep-compression/pruning_2.png)

训练剪枝模型的细节：

1.Regularization: L1会将更多的参数变为0，pruning后仍有很高的精度。L2虽然pruning后精度降低，但是retrain后会有更高的精度。因此论文采用L2 Regularization。caffe默认的就是L2 Regularization。

2.Dropout Ratio Adjustment: 在pruning训练过程中，如果连接被丢弃就没有机会再恢复回来。由于模型参数变得稀疏，减少了over-fitting，因此retraining时dropout的参数要调小。

3.Local Pruning and Parameter Co-adaptation: 

4.Iterative Pruning:

5.Pruning Neurons: Pruning过程中会有0输入或0输出的神经元，这种神经元没有意义，需要删掉。

http://web.stanford.edu/class/ee380/Abstracts/160106.html

![][1]
[1]: http://latex.codecogs.com/gif.latex?\prod%20\(n_{i}\)+1

Next you can update your site name, avatar and other options using the _config.yml file in the root of your repository (shown below).

![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.
