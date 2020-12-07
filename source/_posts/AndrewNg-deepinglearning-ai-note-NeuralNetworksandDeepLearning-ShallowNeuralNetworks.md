---
title: >-
  神经网络和深度学习(Neural Networks and Deep Learning) 第一周 深度学习引言(Introduction to Deep Learning)
date: 2020-09-28 14:56:34
categories: 
- 学习笔记
tags:
- 深度学习
---
吴恩达老师深度学习公开课第一门课————神经网络和深度学习(Neural Networks and Deep Learning) 
**第三周：浅层神经网络(Shallow neural networks)笔记**
<!-- more -->

## 3.1 神经网络概述（Neural Network Overview）
+ **神经网络模型**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001153315103.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001153427154.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001153509649.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)

## 3.2 神经网络的表示（Neural Network Representation）
+ **神经网络的含义**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001153614888.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
（1）输入层：输入特征x1、x2、x3，它们被竖直地堆叠起来
（2）隐藏层:中间的层次，由于中间结点的准确值我们是不知道到的，也就是说你看不见它们在训练集中应具有的值。你能看见输入的值，你也能看见输出的值，但是隐藏层中的东西，在训练集中你是无法看到的。
（3）输出层：负责产生预测值，是神经网络的最后一层。
## 3.3 计算一个神经网络的输出（Computing a Neural Network's output）
+ **神经网络的含义**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001155721623.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020100115591419.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001155957452.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
## 3.4 多样本向量化（Vectorizing across multiple examples）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001160338579.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001160410273.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001160434760.png#pic_center)

## 3.5 向量化实现的解释（Justification for vectorized implementation）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001160803211.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001160832679.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)


## 3.6 激活函数（Activation functions）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001161104499.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
+ **sigmoid激活函数**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001160931767.png#pic_center)
+ **tanh函数（双曲正切函数）**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001161031946.png#pic_center)
+ **ReLu函数**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020100116161550.png#pic_center)
+ **Leaky Relu函数**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020100116165456.png#pic_center)
+ **总结**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020100116172139.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)


## 3.7 为什么需要非线性激活函数？（why need a nonlinear activation function?）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001161828166.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
如果你使用线性激活函数或者没有使用一个激活函数，那么无论你的神经网络有多少层一直在做的只是计算线性函数，所以不如直接去掉全部隐藏层。
不能在隐藏层用线性激活函数，可以用ReLU或者tanh或者leaky ReLU或者其他的非线性激活函数，唯一可以用线性激活函数的通常就是输出层；除了这种情况，会在隐层用线性函数的，除了一些特殊情况。

## 3.8 激活函数的导数（Derivatives of activation functions）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001165304501.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001165317445.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001165328517.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001165339657.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)

## 3.9 神经网络的梯度下降（Gradient descent for neural networks）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001170249503.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)

## 3.10（选修）直观理解反向传播（Backpropagation intuition）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001170249503.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)


## 3.11 随机初始化（Random+Initialization））
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001172911182.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001172931619.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)

