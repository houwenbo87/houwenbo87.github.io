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
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>
\\(C_{i}\\)
为层$$i$$的连接数量，
\\(C_{i0}\\)
为原始网络连接数，
\\(C_{ir}\\)
为retrain后的网络连接数量，
\\(N_{i}\\)
为层$$i$$的神经元数量。
\\(D_{0}\\)
为原始的dropout rate，
\\(D_{r}\\)
为retrain时的dropout rate。

$$C_{i}=N_{i}N_{i-1}$$

$$D_{r}=D_{0}\sqrt{\frac{C_{ir}}{C_{i0}}}$$

3.Local Pruning and Parameter Co-adaptation: 在retrain过程中，使用剪枝后的模型参数作为初始化参数。为了克服网络太深引起的vanishiing gradient问题，作者分段做retrain，比如固定卷积层参数不变，训练全连接。然后再反过来训练。

4.Iterative Pruning: 采用迭代的训练的方式来实现模型剪枝，每次迭代使用贪心策略来寻找最好的连接。

5.Pruning Neurons: Pruning过程中会有0输入或0输出的神经元，这种神经元没有意义，需要删掉。

问题：
1. 剪枝的阈值是如何设定的，论文没有具体介绍。
2. 每层可以剪枝多少连接是否是自动训练的，还是每层设定一个阈值。
3. 如果想实现自动剪枝，每层的剪枝截止的条件该如何设定，剪枝条件是否可以在bp过程中自动调节。
本文的剪枝方法是hard dropconnect，剪掉后就不能再恢复了，剪掉后loss和accuracy都会明显的下降，但是需要迭代好多次才能知道是否能恢复到原来的模型精度。

http://web.stanford.edu/class/ee380/Abstracts/160106.html

![][1]
[1]: http://latex.codecogs.com/gif.latex?\prod%20\(n_{i}\)+1


<img src="http://www.forkosh.com/mathtex.cgi? \Large x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}">


$ J_\alpha(x) = \sum_{m=0}^\infty \frac{(-1)^m}{m! \Gamma (m + \alpha + 1)} {\left({ \frac{x}{2} }\right)}^{2m + \alpha} \text {，行内公式示例} $

$$x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}$$
\\(x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}\\)


Next you can update your site name, avatar and other options using the _config.yml file in the root of your repository (shown below).

![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.
