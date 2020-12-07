---
title: >-
  神经网络和深度学习(Neural Networks and Deep Learning) 第一周 深度学习引言(Introduction to Deep Learning)
date: 2020-09-27 16:21:21
categories: 
- 学习笔记
tags:
- 深度学习
---
吴恩达老师深度学习公开课第一门课————神经网络和深度学习(Neural Networks and Deep Learning) 
**第一周 深度学习引言(Introduction to Deep Learning)笔记**
<!-- more -->

## 1.2 什么是神经网络？(What is a Neural Network) 
![神经网络](https://img-blog.csdnimg.cn/20200927181221920.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
简单的神经网络：x作为输入，通过一个节点（一个单独的神经元），最终输出了y.
从趋近于零开始，然后变成一条直线。这个函数被称作ReLU激活函数，它的全称是Rectified Linear Unit。rectify（修正）可以理解成max(0,x).

![](https://img-blog.csdnimg.cn/2020092718195169.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
起始是样本的每一个特征（x1，x2，x3，x4），通过输入x，神经网络可以得到输出y。样本的特征会以不同的权重，传递给中间的隐藏节点，然后获得一个或多个输出y，在这个例子中就是预测的价格。如果有足够的训练样本，神经网络非常擅长计算从到的精准映射函数。

## 1.3 神经网络的监督学习(Supervised Learning with Neural Networks)
+ **监督学习让神经网络创造了很大的价值**
（1）在线广告投放（通过在网站上输入一个广告的相关信息，因为也输入了用户的信息，于是网站就会考虑是否向用户展示广告）
（2）计算机视觉（输入一个图像，进行图片的分类）
（3）机器翻译（利用神经网络输入英语句子，接着输出一个中文句子）
（4）自动驾驶技术（输入一幅图像，就好像一个信息雷达展示汽车前方有什么，据此，你可以训练一个神经网络，来告诉汽车在马路上面具体的位置）
+ **神经网络的选择**
（1）CNN——多用于图像数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200927185432186.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
（2）RNN——多用于序列数据（虽时间变化的数据，有时间轴）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200927185446801.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
（3）标准神经网络
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020092718541511.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
+ **结构化与非结构化的数据**
（1）结构化数据
数据库中的数据（房价预测中，房子的各种属性）
（2）非结构化数据
音频，原始音频或者你想要识别的图像或文本中的内容（这里的特征可能是图像中的像素值或文本中的单个单词）
## 1.4 为什么深度学习会兴起？(Why is Deep Learning taking off?)
+ **深度学习变热门的原因**
（1）数据规模增大（2）算法的创新（3）计算速度加快
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020092719025281.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
（1）神经网络的规模对结果的影响在数据量小的时候，体现不出来。但是在数据量增多时，神经网络之间的差别将会被放大。
（2）如果要获得更好的性能：要么训练一个更大的神经网络，要么投入更多的数据。但神经网络越复杂，投入的数据越多，最终用于训练的时间将会越长，需要在两者之间进行权衡。
（3）在小的训练集中，各种算法的优先级事实上定义的也不是很明确，所以如果你没有大量的训练集，算法的效果会取决于提取的特征的好坏。但是如果训练集足够大的情况下，算法的效果就会取决于自己设计的神经网络的好坏。
+ **激活函数**
（1）sigmoid函数与ReLU函数
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928110434311.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
（2）sigmoid函数：
在参数接近负无穷时，sigmoid函数的梯度会接近零，所以学习的速度会变得非常缓慢，因为当你实现梯度下降以及梯度接近零的时候，参数会更新的很慢，所以学习的速率也会变的很慢。
（3）ReLU函数：
它的梯度对于所有输入的负值都是零，因此梯度更加不会趋向逐渐减少到零。会使梯度下降的运行速度更快

（4）激活函数的改变，使得代码运行的更快，这也使得我们能够训练规模更大的神经网络，或者是多端口的网络。因为在训练神经网络时，很多时候是凭借直觉的，往往你对神经网络架构有了一个想法，于是你尝试写代码实现你的想法，然后让你运行一个试验环境来告诉你，你的神经网络效果有多好，通过参考这个结果再返回去修改你的神经网络里面的一些细节，然后你不断的重复上面的操作，当你的神经网络需要很长时间去训练，需要很长时间重复这一循环，如果训练时间过长，会让这个周期变得很长。

## 1.5 关于这门课(About this Course)
（1）第一周：关于深度学习的介绍。在每一周的结尾也会有十个多选题用来检验自己对材料的理解；
（2）第二周：关于神经网络的编程知识，了解神经网络的结构，逐步完善算法并思考如何使得神经网络高效地实现。从第二周开始做一些编程训练（付费项目），自己实现算法；
（3）第三周：在学习了神经网络编程的框架之后，你将可以编写一个隐藏层神经网络，所以需要学习所有必须的关键概念来实现神经网络的工作；
（4）第四周：建立一个深层的神经网络。