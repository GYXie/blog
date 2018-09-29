title: 深入AlexNet
categories: Deep Learning
mathjax: true
tags:
  - 深度学习
  - AlexNet
date: 2016-09-21 01:16:55
---

## 素描
- 120万张高分辨率图片
- 1000个不同的类别
- Top-1 错误率37.5%(ImageNet LSVRC-2010)
- Top-5 错误率17.0%(ImageNet LSVRC-2010)
- 6千万参数
- 650,000个神经元
- 5个卷积层
- 3个全连接层, 最后连接一个1000维的softmax
- 非饱和神经元（non-saturating neurons）
- 两块GPU
- dropout
- Top-5错误率15.3%（ILSVRC-2012）
- 两块GTX 580 3GB GPU，训练5、6天
- 


## CNN
- CNN的优势
与拥有类似大小的标准前馈神经网络相比，CNN的连接更少，参数更少，所以更容易训练，而它的最优性能理论上没有太大损失。

## 数据处理
1. ImageNet数据集中的图片分辨率是不一样的，而系统需要固定维度的输入，因为对图片进行下采样（down-sample）成256*256规格。对与长宽不同的图片，先将短的一边放缩到256，再取中间。
2. 对训练集每张图片的每个像素减去图片像素的平均值。

## 架构
- ReLUs(Rectified Linear Units) 非线性特性
- 多块GPU上训练。120万个训练样本训练的模型大到当时的一个GPU无法装下。因为作者把网络分散到了两块GPU。现在的GPUs已经可以很好地实现跨GPU并行，他们可以直接从其他的GPU的内存里读取和写入数据，而不用通过主机内存。
- Local Response Normalization  
  $b^i_{x,y}=a^i_{x,y}/\left(k + \alpha \sum_{j=max(0,i-n/2)}^{min(N-1,i+n/2)}(a^j_{x,y})^2\right)^\beta$
- Overlapping Pooling
  重叠式的Pooling层。
- 总体架构
  有8层，前5层是卷积层，后3层是全连接层。最后一个全连接层的输出发送到1000路的softmax。
  最大化多项式逻辑回归目标函数。

![AlexNet.png](/img/AlexNet.png)
下面是一个网上的AlexNet可视化效果，查看网页可以获得更多信息：https://ethereon.github.io/netscope/#/preset/alexnet
![AlexNet-V.png](/img/AlexNet-V.png)



## 总结
文献[1]是深度学习研究真的必读文章。

*笔者之前Tensorflow+AlexNet做了一个有趣好玩的实验(图片检索），在这里强烈推荐给大家：https://github.com/GYXie/visual-search*


## References
1. Krizhevsky A, Sutskever I, Hinton G E. Imagenet classification with deep convolutional neural networks[C]//Advances in neural information processing systems. 2012: 1097-1105.
2. https://github.com/GYXie/visual-search
3. https://github.com/ethereon/caffe-tensorflow
4. https://ethereon.github.io/netscope/#/preset/alexnet

