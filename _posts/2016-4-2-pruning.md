---
layout: post
title: 阅读笔记 Learning both Weights and Connections for Efficient Neural Networks
---
论文全名 [Learning both Weights and Connections for Efficient Neural Networks](http://arxiv.org/abs/1506.02626)

## Pruning (模型剪枝)
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

阅读后的一些问题：

1. 剪枝的阈值是如何设定的，论文没有具体介绍。

2. 每层可以剪枝多少连接是否是自动训练的，还是需要给每层设定一个阈值。

3. 如果想实现自动剪枝，每层的剪枝截止的条件该如何设定，剪枝条件是否可以在bp过程中自动调节。
本文的剪枝方法是hard dropconnect，剪掉后就不能再恢复了，剪掉后loss和accuracy都会明显的下降，但是需要迭代好多次才能知道是否能恢复到原来的模型精度。

