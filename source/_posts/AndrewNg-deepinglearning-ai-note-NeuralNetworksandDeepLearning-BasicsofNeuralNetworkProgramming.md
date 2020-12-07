---
title: 第二周：神经网络的编程基础(Basics of Neural Network programming)
date: 2020-09-28 11:53:47
categories: 
- 学习笔记
tags:
- 深度学习

mathjax: true
---
吴恩达老师深度学习公开课第一门课————神经网络和深度学习(Neural Networks and Deep Learning) 
**第二周：神经网络的编程基础(Basics of Neural Network programming)**
<!-- more -->
## 2.1 二分类(Binary Classification)
+ **前向传播（forward propagation）和反向传播（backward propagation）**
+ **二分类(binary classification)**
典型算法————逻辑回归
以图片的识别举例，辨别一张图片的内容是否是一只猫。就是将图片的RGB特征提取出来，变成特征向量x，如果图片的像素为64*64，那么RGB每个通道都有64*64位，则向量x的总维度为64*64*3。现在我们用n<sub>x</sub>=12288来表示输入特征向量的维度，有时候为了简洁，我会直接用小写的n来表示输入特征向量的维度。
二分类问题就是习得一个分类器，它以图片的特征向量x作为输入，然后预测输出结果为1还是0，也就是预测图片中是否有猫。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928125317636.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
## 2.2 逻辑回归(Logistic Regression)
（1）给定一个输入特征向量X，它可能对应一张图片，你想识别这张图片识别看它是否是一只猫或者不是一只猫的图片，你想要一个算法能够输出预测$\hat{y}$，也就是你对实际值y的估计。更正式地来说，你想让 $\hat{y}$ 表示y等于1的一种可能性或者是机会，前提条件是给定了输入特征X。
（2）如果X是我们在上个视频看到的图片，你想让 $\hat{y}$ 来告诉你这是一只猫的图片的机率有多大。
（3）我们用$\omega$来表示逻辑回归的参数，这也是一个n<sub>x</sub>维向量（因为$\omega$实际上是特征权重，维度与特征向量相同），参数里面还有b，这是一个实数（表示偏差）。所以给出输入x以及参数$\omega$和b之后，我们怎样产生输出预测值$\hat{y}$，一件你可以尝试却不可行的事是让$\hat{y}$ = $\omega$<sup>T</sup>x+b.
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928130446893.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
(4)问题：$\hat{y}$ = $\omega$<sup>T</sup>x+b应该在0到1之间（$\hat{y}$表示y实际值等于1的机率），但实际的$\hat{y}$可能不在0到1的范围内，所以需要引入sigmoid函数，如果$\hat{y}$输出大于0，则最终的输出大于0.5，如果$\hat{y}$输出小于0，则最终的输出小于0.5。
如果z非常大，则结果会趋近于1，如果z非常小，则结果会趋近于0.
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928133510454.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
## 2.3 逻辑回归的代价函数（Logistic Regression Cost Function）
+ **为什么需要代价函数**
为了训练逻辑回归模型的参数$\omega$和参数b我们，需要一个代价函数，通过训练代价函数来得到参数$\omega$和参数b。先看一下逻辑回归的输出函数：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928134257163.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
为了让模型通过学习调整参数，你需要给予m个样本的训练集，这会让你在训练集上找到参数$\omega$和参数$\{b}$。
对训练集的预测值，我们将它写成$\hat{y}$，我们更希望它会接近于训练集中的$\{y}$值
+ **损失函数**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928134850998.png#pic_center)

我们在逻辑回归中用到的损失函数是：$L(\hat{y},{y})= -{y}{log}{\hat{y}}-{1-}{log}{1-\hat{y}}$

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928135832780.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
+ **代价函数**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928135914607.png#pic_center)

## 2.4 梯度下降法（Gradient Descent）
在你测试集上，通过最小化代价函数（成本函数）${J(\omega,b)}$来训练的参数$\omega$和${b}$，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928140125714.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
朝最陡的下坡方向走一步，不断地迭代,直到走到全局最优解或者接近全局最优解的地方
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928140158947.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928140255262.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
## 2.7 计算图（Computation Graph）
一个神经网络的计算，都是按照前向或反向传播过程组织的。首先我们计算出一个新的网络的输出（前向过程），紧接着进行一个反向传输操作。后者我们用来计算出对应的梯度或导数。计算图解释了为什么我们用这种方式组织这些计算过程。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928140421513.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
## 2.9 逻辑回归中的梯度下降（Logistic Regression Gradient Descent）
怎样通过计算偏导数来实现逻辑回归的梯度下降算法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928140910229.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928141007231.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928141007231.png?x-oss![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928141029550.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
## 2.10  m 个样本的梯度下降(Gradient Descent on m Examples)
```python
J=0;dw1=0;dw2=0;db=0;
for i = 1 to m
    z(i) = wx(i)+b;
    a(i) = sigmoid(z(i));
    J += -[y(i)log(a(i))+(1-y(i)）log(1-a(i));
    dz(i) = a(i)-y(i);
    dw1 += x1(i)dz(i);
    dw2 += x2(i)dz(i);
    db += dz(i);
J/= m;
dw1/= m;
dw2/= m;
db/= m;
w=w-alpha*dw
b=b-alpha*db
```
代码中会有两个for循环，第一个会遍历所有的样本，第二个会遍历一个样本的所有特征。for循环太多，会使代码的执行效率特别低。
解决方案————向量化

## 2.11 向量化(Vectorization)
将特征以及参数都向量化，将循环改编为numpy提供的向量之间的乘法，如np.dot();
## 2.13 向量化逻辑回归(Vectorizing Logistic Regression)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928142834691.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928142915945.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
## 2.14 向量化 logistic 回归的梯度输出（Vectorizing Logistic Regression's Gradient）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928143911260.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
向量化后的运算过程
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928143957113.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
## 2.18 logistic 损失函数的解释（Explanation of logistic regression cost function）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928145334110.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928145354974.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928145412412.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928145427387.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020092814545987.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70#pic_center)

## 课程练习源码
```python
# -*- coding: utf-8 -*-


import numpy as np
import matplotlib.pyplot as plt
import h5py
from data.lr_utils import load_dataset

train_set_x_orig , train_set_y , test_set_x_orig , test_set_y , classes = load_dataset()

m_train = train_set_y.shape[1] #训练集里图片的数量。
m_test = test_set_y.shape[1] #测试集里图片的数量。
num_px = train_set_x_orig.shape[1] #训练、测试集里面的图片的宽度和高度（均为64x64）。

#现在看一看我们加载的东西的具体情况
print ("训练集的数量: m_train = " + str(m_train))
print ("测试集的数量 : m_test = " + str(m_test))
print ("每张图片的宽/高 : num_px = " + str(num_px))
print ("每张图片的大小 : (" + str(num_px) + ", " + str(num_px) + ", 3)")
print ("训练集_图片的维数 : " + str(train_set_x_orig.shape))
print ("训练集_标签的维数 : " + str(train_set_y.shape))
print ("测试集_图片的维数: " + str(test_set_x_orig.shape))
print ("测试集_标签的维数: " + str(test_set_y.shape))

#将训练集的维度降低并转置。
train_set_x_flatten  = train_set_x_orig.reshape(train_set_x_orig.shape[0],-1).T
#将测试集的维度降低并转置。
test_set_x_flatten = test_set_x_orig.reshape(test_set_x_orig.shape[0], -1).T

print ("训练集降维最后的维度： " + str(train_set_x_flatten.shape))
print ("训练集_标签的维数 : " + str(train_set_y.shape))
print ("测试集降维之后的维度: " + str(test_set_x_flatten.shape))
print ("测试集_标签的维数 : " + str(test_set_y.shape))

train_set_x = train_set_x_flatten / 255
test_set_x = test_set_x_flatten / 255

def sigmoid(z):
    """
    参数：
        z  - 任何大小的标量或numpy数组。

    返回：
        s  -  sigmoid（z）
    """
    s = 1 / (1 + np.exp(-z))
    return s

def initialize_with_zeros(dim):
    """
        此函数为w创建一个维度为（dim，1）的0向量，并将b初始化为0。

        参数：
            dim  - 我们想要的w矢量的大小（或者这种情况下的参数数量）

        返回：
            w  - 维度为（dim，1）的初始化向量。
            b  - 初始化的标量（对应于偏差）
    """
    w = np.zeros(shape = (dim,1))
    b = 0
    #使用断言来确保我要的数据是正确的
    assert(w.shape == (dim, 1)) #w的维度是(dim,1)
    assert(isinstance(b, float) or isinstance(b, int)) #b的类型是float或者是int

    return (w , b)

def propagate(w, b, X, Y):
    """
    实现前向和后向传播的成本函数及其梯度。
    参数：
        w  - 权重，大小不等的数组（num_px * num_px * 3，1）
        b  - 偏差，一个标量
        X  - 矩阵类型为（num_px * num_px * 3，训练数量）
        Y  - 真正的“标签”矢量（如果非猫则为0，如果是猫则为1），矩阵维度为(1,训练数据数量)

    返回：
        cost- 逻辑回归的负对数似然成本
        dw  - 相对于w的损失梯度，因此与w相同的形状
        db  - 相对于b的损失梯度，因此与b的形状相同
    """
    m = X.shape[1]

    #正向传播
    A = sigmoid(np.dot(w.T,X) + b) #计算激活值，请参考公式2。
    cost = (- 1 / m) * np.sum(Y * np.log(A) + (1 - Y) * (np.log(1 - A))) #计算成本，请参考公式3和4。

    #反向传播
    dw = (1 / m) * np.dot(X, (A - Y).T) #请参考视频中的偏导公式。
    db = (1 / m) * np.sum(A - Y) #请参考视频中的偏导公式。

    #使用断言确保我的数据是正确的
    assert(dw.shape == w.shape)
    assert(db.dtype == float)
    cost = np.squeeze(cost)
    assert(cost.shape == ())

    #创建一个字典，把dw和db保存起来。
    grads = {
                "dw": dw,
                "db": db
             }
    return (grads , cost)

def optimize(w , b , X , Y , num_iterations , learning_rate , print_cost = False):
    """
    此函数通过运行梯度下降算法来优化w和b

    参数：
        w  - 权重，大小不等的数组（num_px * num_px * 3，1）
        b  - 偏差，一个标量
        X  - 维度为（num_px * num_px * 3，训练数据的数量）的数组。
        Y  - 真正的“标签”矢量（如果非猫则为0，如果是猫则为1），矩阵维度为(1,训练数据的数量)
        num_iterations  - 优化循环的迭代次数
        learning_rate  - 梯度下降更新规则的学习率
        print_cost  - 每100步打印一次损失值

    返回：
        params  - 包含权重w和偏差b的字典
        grads  - 包含权重和偏差相对于成本函数的梯度的字典
        成本 - 优化期间计算的所有成本列表，将用于绘制学习曲线。

    提示：
    我们需要写下两个步骤并遍历它们：
        1）计算当前参数的成本和梯度，使用propagate（）。
        2）使用w和b的梯度下降法则更新参数。
    """

    costs = []

    for i in range(num_iterations):

        grads, cost = propagate(w, b, X, Y)

        dw = grads["dw"]
        db = grads["db"]

        w = w - learning_rate * dw
        b = b - learning_rate * db

        #记录成本
        if i % 100 == 0:
            costs.append(cost)
        #打印成本数据
        if (print_cost) and (i % 100 == 0):
            print("迭代的次数: %i ， 误差值： %f" % (i,cost))

    params  = {
                "w" : w,
                "b" : b }
    grads = {
            "dw": dw,
            "db": db }
    return (params , grads , costs)

def predict(w , b , X ):
    """
    使用学习逻辑回归参数logistic （w，b）预测标签是0还是1，

    参数：
        w  - 权重，大小不等的数组（num_px * num_px * 3，1）
        b  - 偏差，一个标量
        X  - 维度为（num_px * num_px * 3，训练数据的数量）的数据

    返回：
        Y_prediction  - 包含X中所有图片的所有预测【0 | 1】的一个numpy数组（向量）

    """

    m  = X.shape[1] #图片的数量
    Y_prediction = np.zeros((1,m))
    w = w.reshape(X.shape[0],1)

    #计预测猫在图片中出现的概率
    A = sigmoid(np.dot(w.T , X) + b)
    for i in range(A.shape[1]):
        #将概率a [0，i]转换为实际预测p [0，i]
        Y_prediction[0,i] = 1 if A[0,i] > 0.5 else 0
    #使用断言
    assert(Y_prediction.shape == (1,m))

    return Y_prediction

def model(X_train , Y_train , X_test , Y_test , num_iterations = 2000 , learning_rate = 0.5 , print_cost = False):
    """
    通过调用之前实现的函数来构建逻辑回归模型

    参数：
        X_train  - numpy的数组,维度为（num_px * num_px * 3，m_train）的训练集
        Y_train  - numpy的数组,维度为（1，m_train）（矢量）的训练标签集
        X_test   - numpy的数组,维度为（num_px * num_px * 3，m_test）的测试集
        Y_test   - numpy的数组,维度为（1，m_test）的（向量）的测试标签集
        num_iterations  - 表示用于优化参数的迭代次数的超参数
        learning_rate  - 表示optimize（）更新规则中使用的学习速率的超参数
        print_cost  - 设置为true以每100次迭代打印成本

    返回：
        d  - 包含有关模型信息的字典。
    """
    w , b = initialize_with_zeros(X_train.shape[0])

    parameters , grads , costs = optimize(w , b , X_train , Y_train,num_iterations , learning_rate , print_cost)

    #从字典“参数”中检索参数w和b
    w , b = parameters["w"] , parameters["b"]

    #预测测试/训练集的例子
    Y_prediction_test = predict(w , b, X_test)
    Y_prediction_train = predict(w , b, X_train)

    #打印训练后的准确性
    print("训练集准确性："  , format(100 - np.mean(np.abs(Y_prediction_train - Y_train)) * 100) ,"%")
    print("测试集准确性："  , format(100 - np.mean(np.abs(Y_prediction_test - Y_test)) * 100) ,"%")

    d = {
            "costs" : costs,
            "Y_prediction_test" : Y_prediction_test,
            "Y_prediciton_train" : Y_prediction_train,
            "w" : w,
            "b" : b,
            "learning_rate" : learning_rate,
            "num_iterations" : num_iterations }
    return d

d = model(train_set_x, train_set_y, test_set_x, test_set_y, num_iterations = 2000, learning_rate = 0.005, print_cost = True)

#绘制图
costs = np.squeeze(d['costs'])
plt.plot(costs)
plt.ylabel('cost')
plt.xlabel('iterations (per hundreds)')
plt.title("Learning rate =" + str(d["learning_rate"]))
plt.show()

```