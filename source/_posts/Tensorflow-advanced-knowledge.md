---
title: Tensorflow2.0进阶知识
date: 2020-12-03 16:44:59
categories: 
- 深度学习
tags:
- 深度学习框架
- python
- tensorflow
---
Tensorflow2.0的进阶知识
<!-- more -->

# 合并与分割

##  合并
张量的合并可以使用拼接(Concatenate)和堆叠(Stack)操作实现，拼接操作并不会产生新 的维度，仅在现有的维度上合并，而堆叠会创建新维度。

 - **拼接**，通过 tf.concat(tensors, axis)函数拼接张量，其中参数 tensors 保存了所有需要合并的张量 List，axis 参数指定需要合并的维度索引

```python
a = tf.random.normal([10,35,4])
b = tf.random.normal([10,35,4]) 
tf.concat([a,b],axis=2) # 在第三个维度上拼接
```
```python
<tf.Tensor: id=28, shape=(10, 35, 8), dtype=float32, numpy= array([[[-5.13509691e-01, -1.79707789e+00, 6.50747120e-01, ...,2.58447856e-01, 8.47878829e-02, 4.13468748e-01], [-1.17108583e+00, 1.93961406e+00, 1.27830813e-02, ...,
```
```python
a = tf.random.normal([4,32,8])
b = tf.random.normal([6,35,8]) tf.concat([a,b],axis=0) # 非法拼接，其他维度长度不相同
```

 - **堆叠**,拼接操作直接在现有维度上合并数据，并不会创建新的维度。如果在合并数据 时，希望创建一个新的维度，则需要使用 tf.stack 操作。
 - 使用 tf.stack(tensors, axis)可以堆叠方式合并多个张量，通过 tensors 列表表示，参数 axis 指定新维度插入的位置，axis 的用法与 tf.expand_dims 的一致，当axis ≥ 0时，在 axis 之前插入;当axis < 0时，在 axis 之后插入新维度。例如 shape 为[𝑏, 𝑐, h, 𝑤]的张量，在不 同位置通过 stack 操作插入新维度
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201203200253360.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

```python
a = tf.random.normal([35,8])
b = tf.random.normal([35,8])
tf.stack([a,b],axis=0) # 堆叠合并为2个班级，班级维度插入在最前
<tf.Tensor: id=55, shape=(2, 35, 8), dtype=float32, numpy=
array([[[ 3.68728966e-01, -8.54765773e-01, -4.77824420e-01,-3.83714020e-01, -1.73216307e+00, 2.03872994e-02, 2.63810277e+00, -1.12998331e+00],...
```
若选择使用 tf.concat 拼接合并，则：

```python
a = tf.random.normal([35,8])
b = tf.random.normal([35,8])
tf.concat([a,b],axis=0) # 拼接方式合并，没有2个班级的概念
<tf.Tensor: id=108, shape=(70, 8), dtype=float32, numpy=array([[-0.5516891 , -1.5031327 , -0.35369992, 0.31304857, 0.13965549, 0.6696881 , -0.50115544, 0.15550546],[ 0.8622069 , 1.0188094 , 0.18977325, 0.6353301 , 0.05809061,...
```

```python
在这里插入代码片
```

##  分割

 - 合并操作的逆过程就是分割，将一个张量分拆为多个张量。
 - 通过 tf.split(x, num_or_size_splits, axis)可以完成张量的分割操作，参数意义如下:
 - x参数:待分割张量
 - num_or_size_splits参数:切割方案。当num_or_size_splits为单个数值时，如10，表 示等长切割为 10 份;当 num_or_size_splits 为 List 时，List 的每个元素表示每份的长 度，如[2,4,2,2]表示切割为 4 份，每份的长度依次是 2、4、2、2。
 - axis参数:指定分割的维度索引号

```python
x = tf.random.normal([10,35,8])
# 自定义长度的切割，切割为4份，返回4个张量的列表result
result = tf.split(x, num_or_size_splits=[4,2,2,2] ,axis=0)
len(result)
4
```

 - 希望在某个维度上全部按长度为 1 的方式分割，还可以使用 tf.unstack(x, axis)函数。这种方式是 tf.split 的一种特殊情况，切割长度固定为 1，只需要指定切割维度 的索引号即可。

```python
x = tf.random.normal([10,35,8])
result = tf.unstack(x,axis=0) # Unstack为长度为1的张量 
len(result) # 返回10个张量的列表
10
```

#  数据统计
在神经网络的计算过程中，经常需要统计数据的各种属性，如最值、最值位置、均值、范数等信息。由于张量通常较大，直接观察数据很难获得有用信息，通过获取这些张量的统计信息可以较轻松地推测张量数值的分布。
##  向量范数

 - L1范数，定义为向量𝒙的所有元素绝对值之和
```python
x = tf.ones([2,2])
tf.norm(x,ord=1) # 计算L1范数
<tf.Tensor: id=183, shape=(), dtype=float32, numpy=4.0>
```
 - L2范数，定义为向量𝒙的所有元素的平方和，再开根号

```python
tf.norm(x,ord=2) # 计算L2范数
<tf.Tensor: id=189, shape=(), dtype=float32, numpy=2.0>
```

 - ∞−范数，定义为向量𝒙的所有元素绝对值的最大值:

```python
import numpy as np
tf.norm(x,ord=np.inf) # 计算∞范数
<tf.Tensor: id=194, shape=(), dtype=float32, numpy=1.0>
```


##  最值、均值、和

 - 通过 tf.reduce_max、tf.reduce_min、tf.reduce_mean、tf.reduce_sum
   函数可以求解张量在某个维度上的最大、最小、均值、和，也可以求全局最大、最小、均值、和信息。

```python
tf.reduce_min(x,axis=1) # 统计概率维度上的最小值
<tf.Tensor: id=206, shape=(4,), dtype=float32, numpy=array([-
0.27862206, -2.4480672 , -1.9983795 , -1.5287997 ], dtype=float32)>
```

```python
x = tf.random.normal([4,10])
# 统计全局的最大、最小、均值、和，返回的张量均为标量 
tf.reduce_max(x),tf.reduce_min(x),tf.reduce_mean(x)
(<tf.Tensor: id=218, shape=(), dtype=float32, numpy=1.8653786>,
 <tf.Tensor: id=220, shape=(), dtype=float32, numpy=-1.9751656>,
 <tf.Tensor: id=222, shape=(), dtype=float32, numpy=0.014772797>)
```

 - 通过 tf.argmax(x, axis)和 tf.argmin(x, axis)可以求解在 axis 轴上，x 的最大值、最小值所
   在的索引号

```python
out = tf.random.normal([2,10])
out = tf.nn.softmax(out, axis=1) # 通过softmax函数转换为概率值
pred = tf.argmax(out, axis=1) # 选取概率最大的位置 pred
<tf.Tensor: id=262, shape=(2,), dtype=int64, numpy=array([0, 0],dtype=int64)>
```

#  张量比较
为了计算分类任务的准确率等指标，一般需要将预测结果和真实标签比较，统计比较 结果中正确的数量来计算准确率。

 - 通过 tf.argmax 获取预测类别

```python
out = tf.random.normal([100,10])
out = tf.nn.softmax(out, axis=1) # 输出转换为概率 
pred = tf.argmax(out, axis=1) # 计算预测值
# 模型生成真实标签
y = tf.random.uniform([100],dtype=tf.int64,maxval=10)
out = tf.equal(pred,y) # 预测值与真实值比较，返回布尔类型的张量
out = tf.cast(out, dtype=tf.float32) # 布尔型转int型 
correct = tf.reduce_sum(out) # 统计True的个数
<tf.Tensor: id=293, shape=(), dtype=float32, numpy=12.0>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201203213935313.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

#  填充与复制
##  填充

 - 对于图片数据的高和宽、序列信号的长度，维度长度可能各不相同。为了方便网络的 并行计算，需要将不同长度的数据扩张为相同长度。
 - 填充操作可以通过 tf.pad(x, paddings)函数实现，参数 paddings 是包含了多个 [Left Padding,Right Padding]的嵌套方案 List，如[[0,0], [2,1], [1,2]]表示第一个维度不填
   充，第二个维度左边(起始处)填充两个单元，右边(结束处)填充一个单元，第三个维度左边 填充一个单元，右边填充两个单元。

```python
a = tf.constant([1,2,3,4,5,6]) # 第一个句子 b = tf.constant([7,8,1,6]) # 第二个句子
b = tf.pad(b, [[0,2]]) # 句子末尾填充 2 个 0
b # 填充后的结果
<tf.Tensor: id=3, shape=(6,), dtype=int32, numpy=array([7, 8, 1, 6,0, 0])>
```

 - 对于小于 80 个单词的句子，在末尾填充 相应数量的 0;对大于 80 个单词的句子，截断超过规定长度的部分单词

```python
total_words = 10000 # 设定词汇量大小 max_review_len = 80 # 最大句子长度 embedding_len = 100 # 词向量长度
# 加载IMDB数据集
(x_train, y_train), (x_test, y_test) =
keras.datasets.imdb.load_data(num_words=total_words)
# 将句子填充或截断到相同长度，设置为末尾填充和末尾截断方式
x_train = keras.preprocessing.sequence.pad_sequences(x_train,
maxlen=max_review_len,truncating='post',padding='post')
x_test = keras.preprocessing.sequence.pad_sequences(x_test,
maxlen=max_review_len,truncating='post',padding='post') print(x_train.shape, x_test.shape) # 打印等长的句子张量形状
```

 - 以 28 × 28大小的图片数据为例，如果网络层所接受的数据高宽为32 × 32，则必须将28 × 28 大小填充到32 × 32，可以选择在图片矩阵的上、下、左、右方向各填充 2 个单元

```python
x = tf.random.normal([4,28,28,1])
# 图片上下、左右各填充 2 个单元 tf.pad(x,[[0,0],[2,2],[2,2],[0,0]])
```

##  复制

 - 通过 tf.tile 函数可以在任意维度将数据重复复制多份，如 shape 为[4,32,32,3]的数据， 复制方案为
   multiples=[2,3,3,1]，即通道数据不复制，高和宽方向分别复制 2 份，图片数再 复制 1 份

```python
x = tf.random.normal([4,32,32,3]) 
tf.tile(x,[2,3,3,1]) # 数据复制
<tf.Tensor: id=25, shape=(8, 96, 96, 3), dtype=float32, numpy= array([[[[ 1.20957184e+00, 2.82766962e+00, 1.65782201e+00],[ 3.85402292e-01, 2.00732923e+00, -2.79068202e-01],
```
#  数据限幅

 - 通过 tf.maximum(x, a)实现数据的下限幅，即𝑥 ∈ [𝑎, +∞);
 - 通过 tf.minimum(x, a)实现数据的上限幅，即𝑥 ∈ (−∞, 𝑎]

```python
x = tf.range(9) 
tf.maximum(x,2) # 下限幅到 2
<tf.Tensor: id=48, shape=(9,), dtype=int32, numpy=array([2, 2, 2, 3,4, 5, 6, 7, 8])>
tf.minimum(x,7) # 上限幅到 7
<tf.Tensor: id=41, shape=(9,), dtype=int32, numpy=array([0, 1, 2, 3,4, 5, 6, 7, 7])>
```

 - 使用 tf.clip_by_value 函数实现上下限幅

```python
x = tf.range(9) 
tf.clip_by_value(x,2,7) # 限幅为 2~7
<tf.Tensor: id=66, shape=(9,), dtype=int32, numpy=array([2, 2, 2, 3,4, 5, 6, 7, 7])>
```

#  高级操作
##  tf.gather

 - tf.gather 可以实现根据索引号收集数据的目的
```python
# 可以收集索引号[0,1]，并指定维度axis = 0 
tf.gather(x,[0,1],axis=0)
# 可以收集索引号[0,3,8,11,12,26]，并指定维度axis = 1
tf.gather(x,[0,3,8,11,12,26],axis=1)
#索引号可以乱序排放
tf.gather(a,[3,1,0,2],axis=0) # 收集第 4,2,1,3 号元素
```
##  tf.gather_nd
 - 通过 tf.gather_nd 函数，可以通过指定每次采样点的多维坐标来实现采样多个点的目的。

```python
# 根据多维坐标收集数据 
tf.gather_nd(x,[[1,1],[2,2],[3,3]])
<tf.Tensor: id=256, shape=(3, 8), dtype=int32, numpy= array([[45, 34, 99, 17, 3, 1, 43, 86],[11, 25, 84, 95, 97, 95, 69, 69],[ 0, 89, 52, 29, 76, 7, 2, 98]])>
```

```python
# 根据多维度坐标收集数据 
tf.gather_nd(x,[[1,1,2],[2,2,3],[3,3,4]])
<tf.Tensor: id=259, shape=(3,), dtype=int32, numpy=array([99, 95,76])>
```

##  tf.boolean_mask

 - 除了可以通过给定索引号的方式采样，还可以通过给定掩码(Mask)的方式进行采样。**掩码的长度必须与对应维度的长度一致**

```python
# 根据掩码方式采样，给出掩码和维度索引 
tf.boolean_mask(x,mask=[True, False,False,True],axis=0)
```

 -  tf.boolean_mask 的用法其实与 tf.gather 非常类似，只不过一个通过掩码 方式采样，一个直接给出索引号采样

```python
x = tf.random.uniform([2,3,8],maxval=100,dtype=tf.int32)
tf.gather_nd(x,[[0,0],[0,1],[1,1],[1,2]]) # 多维坐标采集
<tf.Tensor: id=325, shape=(4, 8), dtype=int32, numpy= array([[52, 81, 78, 21, 50, 6, 68, 19],
[53, 70, 62, 12, 7, 68, 36, 84], [62, 30, 52, 60, 10, 93, 33, 6], [97, 92, 59, 87, 86, 49, 47, 11]])>
```

```python
# 多维掩码采样 
tf.boolean_mask(x,[[True,True,False],[False,True,True]])
<tf.Tensor: id=354, shape=(4, 8), dtype=int32, numpy= array([[52, 81, 78, 21, 50, 6, 68, 19],
[53, 70, 62, 12, 7, 68, 36, 84],
[62, 30, 52, 60, 10, 93, 33, 6], [97, 92, 59, 87, 86, 49, 47, 11]])>
```

##   tf.where

 - 通过 tf.where(cond, a, b)操作可以根据 cond 条件的真假从参数𝑨或𝑩中读取数据
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201203220631843.png)

```python
a = tf.ones([3,3]) # 构造 a 为全 1 矩阵
b = tf.zeros([3,3]) # 构造 b 为全 0 矩阵
# 构造采样条件
cond =
tf.constant([[True,False,False],[False,True,False],[True,True,False]])
tf.where(cond,a,b) # 根据条件从 a,b 中采样
<tf.Tensor: id=384, shape=(3, 3), dtype=float32, numpy=
array([[1., 0., 0.],
      [0., 1., 0.],
[1., 1., 0.]], dtype=float32)>
#可以看到，返回的张量中为 1 的位置全部来自张量 a，返回的张量中为 0 的位置来自张量 b。
```

 - 我们需要提取张量中所有正数的数据和索引。 首先构造张量 a，并通过比较运算得到所有正数的位置掩码：
 

```python
x = tf.random.normal([3,3]) # 构造 a
mask=x>0 # 比较操作，等同于 tf.math.greater()  通过比较运算，得到所有正数的掩码
indices=tf.where(mask) # 提取所有大于 0 的元素索引
tf.gather_nd(x,indices) # 提取正数的元素值
```
或者

```python
tf.boolean_mask(x,mask) # 通过掩码提取正数的元素值
```

##  scatter_nd

 - 通过 tf.scatter_nd(indices, updates, shape)函数可以高效地刷新张量的部分数据，但是这 个函数只能在全 0 的白板张量上面执行刷新操作，因此可能需要结合其它操作来实现现有 张量的数据刷新功能。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201203221154114.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

```python
# 构造需要刷新数据的位置参数，即为 4、3、1 和 7 号位置 
indices = tf.constant([[4], [3], [1], [7]])
# 构造需要写入的数据，4 号位写入 4.4,3 号位写入 3.3，以此类推
updates = tf.constant([4.4, 3.3, 1.1, 7.7])
# 在长度为 8 的全 0 向量上根据 indices 写入 updates 数据 
tf.scatter_nd(indices, updates, [8])
```

##  meshgrid

 - 通过 tf.meshgrid 函数可以方便地生成二维网格的采样点坐标，方便可视化等应用场
合。考虑2个自变量x和y的Sinc函数表达式为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201203221449852.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201203221838187.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)


#  经典数据集加载
在 TensorFlow 中，keras.datasets 模块提供了常用经典数据集的自动下载、管理、加载 与转换功能，并且提供了 tf.data.Dataset 数据集对象，方便实现多线程(Multi-threading)、预 处理(Preprocessing)、随机打散(Shuffle)和批训练(Training on Batch)等常用数据集的功能。

 - Boston Housing，波士顿房价趋势数据集，用于回归模型训练与测试。
 - CIFAR10/100，真实图片数据集，用于图片分类任务。
 - MNIST/Fashion_MNIST，手写数字图片数据集，用于图片分类任务。
 - IMDB，情感分类任务数据集，用于文本分类任务。

```python
#自动加载 MNIST 数据集
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import datasets # 导入经典数据集加载模块
# 加载 MNIST 数据集
(x, y), (x_test, y_test) = datasets.mnist.load_data()
print('x:', x.shape, 'y:', y.shape, 'x test:', x_test.shape, 'y test:',
y_test)
# 返回数组的形状
x: (60000, 28, 28) y: (60000,) x test: (10000, 28, 28) y test: [7 2 1 ... 4
5 6]

#数据加载进入内存后，需要转换成 Dataset 对象，才能利用 TensorFlow 提供的各种便 捷功能。
#通过 Dataset.from_tensor_slices 可以将训练部分的数据图片 x 和标签 y 都转换成 Dataset 对象:
train_db = tf.data.Dataset.from_tensor_slices((x, y)) # 构建 Dataset 对象
```

##  随机打散

 - 通过 Dataset.shuffle(buffer_size)工具可以设置 Dataset 对象随机打散数据之间的顺序
 - buffer_size 参数指定缓冲池的大小，一般设置为一个较大的常数即可

```python
train_db = train_db.shuffle(10000) # 随机打散样本，不会打乱样本与标签映射关系
```

##  批训练

 - 为了一次能够从 Dataset 中产生 Batch Size 数量的样本，需要设置 Dataset 为批训练方式

```python
train_db = train_db.batch(128) # 设置批训练，batch size 为 128
```

 - Batch Size 一般根据用户 的 GPU 显存资源来设置，当显存不足时，可以适量减少 Batch Size 来减少算法的显存使用量。

##  预处理

 - 从 keras.datasets 中加载的数据集的格式大部分情况都不能直接满足模型的输入要求， 因此需要根据用户的逻辑自行实现预处理步骤。Dataset 对象通过提供 map(func)工具函 数，可以非常方便地调用用户自定义的预处理逻辑，它实现在 func 函数里。
 
```python
# 预处理函数实现在 preprocess 函数中，传入函数名即可 
train_db = train_db.map(preprocess)



#将 MNIST 图片数据映射到𝑥 ∈ [0,1]区间，视图调整为 [𝑏, 28 ∗ 28];对于标签数据，
#我们选择在预处理函数里面进行 One-hot 编码。preprocess 函 数实现如下:
def preprocess(x, y): # 自定义的预处理函数
# 调用此函数时会自动传入 x,y 对象，shape 为[b, 28, 28], [b] # 标准化到 0~1
	x = tf.cast(x, dtype=tf.float32) / 255.
	x = tf.reshape(x, [-1, 28*28])
	y = tf.cast(y, dtype=tf.int32)
	y = tf.one_hot(y, depth=10)
	# 打平
	# 转成整型张量
	# one-hot编码
	# 返回的 x,y 将替换传入的 x,y 参数，从而实现数据的预处理功能 return x,y
```

##  循环训练

 - 对于Dataset对象进行迭代

```python
for step, (x,y) in enumerate(train_db): # 迭代数据集对象，带 step 参数
for x,y in train_db: # 迭代数据集对象
```

 - 完成一个Batch的数据训练叫做一个Step，通过多个step来完成整个训练集的一次迭代，叫做一个Epoch。

```python
for epoch in range(20): # 训练 Epoch 数
	for step, (x,y) in enumerate(train_db): # 迭代 Step 数
# training...
```
 - 可以通过设置 Dataset 对象，使得数据集对象内部遍历多次才会退出
```python
train_db = train_db.repeat(20) # 数据集迭代 20 遍才终止
```

#  MNIST 测试实战

```python
#%%
import  matplotlib
from    matplotlib import pyplot as plt
# Default parameters for plots
matplotlib.rcParams['font.size'] = 20
matplotlib.rcParams['figure.titlesize'] = 20
matplotlib.rcParams['figure.figsize'] = [9, 7]
matplotlib.rcParams['font.family'] = ['STKaiTi']
matplotlib.rcParams['axes.unicode_minus']=False
import  tensorflow as tf
from    tensorflow import keras
from    tensorflow.keras import datasets, layers, optimizers
import  os





os.environ['TF_CPP_MIN_LOG_LEVEL']='2'
print(tf.__version__)
# 自定义的预处理函数
# 调用此函数时会自动传入 x,y 对象，shape 为[b, 28, 28], [b] # 标准化到 0~1
# 打平
	# 转成整型张量
	# one-hot编码
	# 返回的 x,y 将替换传入的 x,y 参数，从而实现数据的预处理功能
def preprocess(x, y):
    # [b, 28, 28], [b]
    print(x.shape,y.shape)
    x = tf.cast(x, dtype=tf.float32) / 255.
    x = tf.reshape(x, [-1, 28*28])
    y = tf.cast(y, dtype=tf.int32)
    y = tf.one_hot(y, depth=10)

    return x,y


#加载MNIST数据集
(x, y), (x_test, y_test) = datasets.mnist.load_data()
print('x:', x.shape, 'y:', y.shape, 'x test:', x_test.shape, 'y test:', y_test)
#batch_size
batchsz = 512
#通过 Dataset.from_tensor_slices 可以将训练部分的数据图片 x 和标签 y 都转换成 Dataset 对象:
train_db = tf.data.Dataset.from_tensor_slices((x, y))
#将数据集打乱
train_db = train_db.shuffle(1000)
#规定批处理的大小
train_db = train_db.batch(batchsz)
#添加数据预处理的函数
train_db = train_db.map(preprocess)
#数据会迭代20个epoch
train_db = train_db.repeat(20)

#将测试集的图片x和标签y转换成Dataset对象
test_db = tf.data.Dataset.from_tensor_slices((x_test, y_test))
#与之前相同的操作
test_db = test_db.shuffle(1000).batch(batchsz)
x,y = next(iter(train_db))
print('train sample:', x.shape, y.shape)
# print(x[0], y[0])




#%%
def main():

    # learning rate
    lr = 1e-2
    accs,losses = [], []


    # 784 => 512
    w1, b1 = tf.Variable(tf.random.normal([784, 256], stddev=0.1)), tf.Variable(tf.zeros([256]))
    # 512 => 256
    w2, b2 = tf.Variable(tf.random.normal([256, 128], stddev=0.1)), tf.Variable(tf.zeros([128]))
    # 256 => 10
    w3, b3 = tf.Variable(tf.random.normal([128, 10], stddev=0.1)), tf.Variable(tf.zeros([10]))





    for step, (x,y) in enumerate(train_db):

        # [b, 28, 28] => [b, 784]
        x = tf.reshape(x, (-1, 784))

        with tf.GradientTape() as tape:

            # layer1.
            h1 = x @ w1 + b1
            h1 = tf.nn.relu(h1)
            # layer2
            h2 = h1 @ w2 + b2
            h2 = tf.nn.relu(h2)
            # output
            out = h2 @ w3 + b3
            # out = tf.nn.relu(out)

            # compute loss
            # [b, 10] - [b, 10]
            #损失函数
            loss = tf.square(y-out)
            # [b, 10] => scalar
            #每一轮的平均loss
            loss = tf.reduce_mean(loss)

        #进行梯度的更新
        grads = tape.gradient(loss, [w1, b1, w2, b2, w3, b3])
        for p, g in zip([w1, b1, w2, b2, w3, b3], grads):
            p.assign_sub(lr * g)


        # print
        if step % 80 == 0:
            print(step, 'loss:', float(loss))
            losses.append(float(loss))
        #在测试集上计算准确率
        if step %80 == 0:
            # evaluate/test
            total, total_correct = 0., 0

            for x, y in test_db:
                # layer1.
                h1 = x @ w1 + b1
                h1 = tf.nn.relu(h1)
                # layer2
                h2 = h1 @ w2 + b2
                h2 = tf.nn.relu(h2)
                # output
                out = h2 @ w3 + b3
                # [b, 10] => [b]
                pred = tf.argmax(out, axis=1)
                # convert one_hot y to number y
                y = tf.argmax(y, axis=1)
                # bool type
                correct = tf.equal(pred, y)
                # bool tensor => int tensor => numpy
                total_correct += tf.reduce_sum(tf.cast(correct, dtype=tf.int32)).numpy()
                total += x.shape[0]

            print(step, 'Evaluate Acc:', total_correct/total)

            accs.append(total_correct/total)


    plt.figure()
    x = [i*80 for i in range(len(losses))]
    plt.plot(x, losses, color='C0', marker='s', label='训练')
    plt.ylabel('MSE')
    plt.xlabel('Step')
    plt.legend()
    plt.savefig('train.svg')

    plt.figure()
    plt.plot(x, accs, color='C1', marker='s', label='测试')
    plt.ylabel('准确率')
    plt.xlabel('Step')
    plt.legend()
    plt.savefig('test.svg')

if __name__ == '__main__':
    main()
```
