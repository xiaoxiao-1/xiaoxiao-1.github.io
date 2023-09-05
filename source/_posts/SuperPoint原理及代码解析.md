---
title: SuperPoint原理及代码解析
date: 2023-08-11 17:55:01
description: 
categories: 
- 论文阅读
tags:
- 深度学习
- 计算机视觉
- slam
mathjax: true
---

SuperPoint原理及代码解析

首先列出两个个人觉得写的比较好的博客：

[语义SLAM | 深度学习用于特征提取 : SuperPoint（三）](https://zhuanlan.zhihu.com/p/69515306)

[笔记：SuperPoint: Self-Supervised Interest Point Detection and Description 自监督深度学习特征点](https://vincentqin.gitee.io/posts/superpoint/)



# SuperPoint系列

SuperPoint系列第一篇是https://arxiv.org/abs/1606.03798，这篇文章的主要的贡献是使用卷积神经网络提取特征，然后通过全连接层拟合单应性矩阵，显示出了不错的效果。这篇文章还提到直接去拟合单应性矩阵是困难的，所以他们提到了另外一种替代方法，

![image-20230811183030502](/home/devil/Documents/Hexo_Blog/xiaoxiao-1.github.io/source/_posts/SuperPoint原理及代码解析/image-20230811183030502.png)

上式代表像素坐标的偏移量，我们一旦知道四个点的对应关系，单应性矩阵也可以得到，所以这两种表示方法是一一对应的。上面这篇文章的网络结构如下：

![image-20230811183359571](/home/devil/Documents/Hexo_Blog/xiaoxiao-1.github.io/source/_posts/SuperPoint原理及代码解析/image-20230811183359571.png)



SuperPoint系列第二篇文章是[[Toward Geometric Deep SLAM]](https://arxiv.org/pdf/1707.07410.pdf)，这篇文章开始初步具备特征点检测的雏形，也为SuperPoint的诞生做了铺垫。首先这篇提出了MagicPoint和MagicWrap两个网络结构，前者用于提取图片中像素是特征点的概率，后者输入是已经提取过特征点的图片，输出是两张图片的相对位姿。网络结构如下：

![image-20230811194447281](/home/devil/Documents/Hexo_Blog/xiaoxiao-1.github.io/source/_posts/SuperPoint原理及代码解析/image-20230811194447281.png)

其中MagicPoint网络的结构为：

![image-20230811194520828](/home/devil/Documents/Hexo_Blog/xiaoxiao-1.github.io/source/_posts/SuperPoint原理及代码解析/image-20230811194520828.png)

使用类似VGG的网络结构作为编码器，得到原图1/8大小，65（8*8+1）通道的特征图。这里使用了亚像素卷积的形式，因为长宽为原图的1/8，但是我们希望得到每一个像素作为特征点的概率，一种方法是反卷积，但是其他博客中说反卷积会引入过多的认为因素并且会出现棋盘格效应，所以这里采用了亚像素卷积。

[亚像素卷积解释](https://blog.csdn.net/leviopku/article/details/84975282)

那另外一个通道是怎么回事呢？因为我们不能保证任何一个8*8的block中都有特征点，但是如果我们要做softmax就很大概率会出现在没有特征点的区域中也出现了特征点。所以这里多了一个通道，这个通道参与softmax运算以均衡其他通道。

MagicWrap网络的结构为：

