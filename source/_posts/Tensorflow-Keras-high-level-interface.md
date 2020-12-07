---
title: Tensorflow中Keras 高层接口
date: 2020-12-04 13:41:13
categories: 
- 深度学习
tags:
- 深度学习框架
- python
- tensorflow
---
TensorFlow中的子模块 tf.keras中快速构建神经网络的高层接口
<!-- more -->

#  常见功能模块
**Keras 提供了一系列高层的神经网络相关类和函数，如经典数据集加载函数、网络层类、模型容器、损失函数类、优化器类、经典模型类。**

##  常见网络层类

 - 对于常见的神经网络层，可以使用张量方式的底层接口函数来实现，这些接口函数一 般在 tf.nn 模块中
 - 对于常见的网络层，我们一般直接使用层方式来完成模型的 搭建，在 tf.keras.layers 命名空间中提供了大量常见网络层的类，如全连接层、激活函数层、池化层、卷积层、循环神经网络层。
 - 例如以 Softmax 层为例，它既可以使用 tf.nn.softmax 函数在前向传播逻辑中完成 Softmax 运算，也可以通过 layers.Softmax(axis)类搭建 Softmax 网络层
```python
import tensorflow as tf
# 导入keras模型，不能使用import keras，它导入的是标准的Keras库 
from tensorflow import keras
from tensorflow.keras import layers # 导入常见网络层类
#然后创建 Softmax 层，并调用__call__方法完成前向计算
x = tf.constant([2.,1.,0.1]) # 创建输入张量 layer = layers.Softmax(axis=-1) # 创建Softmax层
out = layer(x) # 调用softmax前向计算，输出为out
#可以直接通过 tf.nn.softmax()函数完成计算
out = tf.nn.softmax(x) # 调用 softmax 函数完成前向计算
```
##  网络容器
为了避免当网络层数较深时，手动调用每一层的类实例完成前向传播运算的麻烦。
 - 可以通过 Keras 提供的网络容器 Sequential 将多个
   网络层封装成一个大网络模型，只需要调用网络模型的实例一次即可完成数据从第一层到 最末层的顺序传播运算。

```python
# 导入Sequential容器
from tensorflow.keras 
import layers, Sequential 
network = Sequential([ # 封装为一个网络
layers.Dense(3, activation=None), # 全连接层，此处不使用激活函数 layers.ReLU(),#激活函数层
layers.Dense(2, activation=None), # 全连接层，此处不使用激活函数
layers.ReLU() #激活函数层 
])
x = tf.random.normal([4,3])
out = network(x) # 输入从第一层开始，逐层传播至输出层，并返回输出层的输出
```

 - Sequential 容器也可以通过 add()方法继续追加新的网络层，实现动态创建网络的功能

```python
layers_num = 2 # 堆叠2次
network = Sequential([]) # 先创建空的网络容器 for _ in range(layers_num):
network.add(layers.Dense(3)) # 添加全连接层
network.add(layers.ReLU())# 添加激活函数层 
#通过调用类的 build 方法并指定 输入大小，即可自动创建所有层的内部张量
network.build(input_shape=(4, 4)) # 创建网络参数  
network.summary()
```

 - 当我们通过 Sequential 容量封装多个网络层时，每层的参数列表将会自动并入 Sequential 容器的参数列表中，不需要人为合并网络参数列表。Sequential 对象的 trainable_variables 和 variables 包含了所有层的待优化张量列表 和全部张量列表。

```python
# 打印网络的待优化参数名与shape
for p in network.trainable_variables: 
	print(p.name, p.shape) # 参数名和形状
```
#  模型装配、训练与测试
##  模型装配

 - 在 Keras 中，有 2 个比较特殊的类:keras.Model 和 keras.layers.Layer 类。
 - Layer 类是网络层的母类，定义了网络层的一些常见功能，如添加权值、管理权值列表等。
 - Model 类是网络的母类，除了具有 Layer 类的功能，还添加了保存模型、加载模型、训练与测试模型等便捷功能。
 - Sequential也是 Model 的子类，因此具有 Model 类的所有功能。

 -创建5层的全连接网络

```python
# 创建5层的全连接网络
network = Sequential([layers.Dense(256, activation='relu'),
                 layers.Dense(128, activation='relu'),
                 layers.Dense(64, activation='relu'),
layers.Dense(32, activation='relu'),
                 layers.Dense(10)])
network.build(input_shape=(4, 28*28))
network.summary()
```

 - Keras 中提供了 compile()和 fit()函数，创建网络后，正常的流程是循环迭代数据集多个 Epoch，每次按批产生训练数据、前向计 算，然后通过损失函数计算误差值，并反向传播自动计算梯度、更新网络参数。

```python
# 导入优化器，损失函数模块
from tensorflow.keras import optimizers,losses 
# 模型装配
# 采用Adam优化器，学习率为0.01;采用交叉熵损失函数，包含Softmax 
network.compile(optimizer=optimizers.Adam(lr=0.01),
loss=losses.CategoricalCrossentropy(from_logits=True),
metrics=['accuracy'] # 设置测量指标为准确率 )
```
##  模型训练
 - 模型装配完成后，即可通过 fit()函数送入待训练的数据集和验证用的数据集

```python
# 指定训练集为train_db，验证集为val_db,训练5个epochs，每2个epoch验证一次 # 返回训练轨迹信息保存在history对象中
history = network.fit(train_db, epochs=5, validation_data=val_db,
validation_freq=2)
```
 train_db 为 tf.data.Dataset 对象，也可以传入 Numpy Array 类型的数据;epochs 参数指 定训练迭代的 Epoch 数量;validation_data 参数指定用于验证(测试)的数据集和验证的频率 validation_freq

 - fit 函数会返回训练过程的数据记录 history，其中 history.history 为字典对象，包含了训练过程中的 loss、测量指标等记录项

```python
history.history # 打印训练记录
```
fit()函数的运行代表了网络的训练过程，因此会消耗相当的训练时间，并在训练结束 后才返回，训练中产生的历史数据可以通过返回值对象取得。

##  模型测试

 - 通过 Model.predict(x)方法即可完成模型的预测

```python
# 加载一个 batch 的测试数据
x,y = next(iter(db_test))
print('predict x:', x.shape) # 打印当前batch的形状
out = network.predict(x) # 模型预测，预测结果保存在out中
print(out)
```

 - 如果只是简单的测试模型的性能，可以通过 Model.evaluate(db)循环测试完 db 数据集 上所有样本，并打印出性能指标

```python
network.evaluate(db_test) # 模型测试，测试在db_test上的性能表现
```
#  模型保存与加载
模型训练完成后，需要将模型保存到文件系统上，从而方便后续的模型测试与部署工作。

##  张量方式

 - 通过调用 Model.save_weights(path)方法即可将当前的 网络参数保存到 path 文件上
```python
network.save_weights('weights.ckpt') # 保存模型的所有张量数据
```
将 network 模型保存到 weights.ckpt 文件上。
 - 在需要的时候，先创建好网络对象， 然后调用网络对象的 load_weights(path)方法即可将指定的模型文件中保存的张量数值写入 到当前网络参数中去

```python
# 保存模型参数到文件上 
network.save_weights('weights.ckpt') 
print('saved weights.')
del network # 删除网络对象
# 重新创建相同的网络结构
network = Sequential([layers.Dense(256, activation='relu'),
layers.Dense(128, activation='relu'),
layers.Dense(64, activation='relu'),
layers.Dense(32, activation='relu'),
layers.Dense(10)])
network.compile(optimizer=optimizers.Adam(lr=0.01),
        loss=tf.losses.CategoricalCrossentropy(from_logits=True),
        metrics=['accuracy']
   )
# 从参数文件中读取数据并写入当前网络 
network.load_weights('weights.ckpt')
print('loaded weights!')
```

##  网络方式

 - 通过 Model.save(path)函数可以将模型的结构以及模型的参数保存到 path 文件上，在不 需要网络源文件的条件下，通过 keras.models.load_model(path)即可恢复网络结构和网络参数

```python
# 保存模型结构与模型参数到文件
network.save('model.h5')
print('saved total model.')
del network # 删除网络对象
```

 - 通过 model.h5 文件即可恢复出网络的结构和状态，不需要提前创建网络对象

```python
 # 从文件恢复网络结构与网络参数
network = keras.models.load_model('model.h5')
```
model.h5 文件除了保存了模型参数外，还应保存了网络结构信息，不需要提前 创建模型即可直接从文件中恢复出网络 network 对象

##  SavedModel 方式

 - 通过 tf.saved_model.save (network, path)即可将模型以 SavedModel 方式保存到 path 目录中

```python
# 保存模型结构与模型参数到文件
tf.saved_model.save(network, 'model-savedmodel')
print('saving savedmodel.')
del network # 删除网络对象
```

 - 通过 tf.saved_model.load 函数即可恢复出模型对象，我们在恢复出模型实例后，完成测试准确率的计算

```python
print('load savedmodel from file.')
# 从文件恢复网络结构与网络参数
network = tf.saved_model.load('model-savedmodel') # 准确率计量器
acc_meter = metrics.CategoricalAccuracy()
for x,y in ds_val: # 遍历测试集
pred = network(x) # 前向计算
acc_meter.update_state(y_true=y, y_pred=pred) # 更新准确率统计 # 打印准确率
print("Test Accuracy:%f" % acc_meter.result())
```
#  自定义网络
对于需要创建自定义逻辑的网络层，可以通过自定义类来实现。在创建自定义网络层 类时，需要继承自 layers.Layer 基类;创建自定义的网络类时，需要继承自 keras.Model 基 类，这样建立的自定义类才能够方便的利用 Layer/Model 基类提供的参数管理等功能，同 时也能够与其他的标准网络层类交互使用。
##  自定义网络层

 - 对于自定义的网络层，至少需要实现初始化__init__方法和前向传播逻辑 call 方法。
 - 首先创建类，并继承自 Layer 基类。创建初始化方法，并调用母类的初始化函数，由 于是全连接层，因此需要设置两个参数:输入特征的长度 inp_dim 和输出特征的长度 outp_dim，并通过 self.add_variable(name, shape)创建 shape 大小，名字为 name 的张量𝑾， 并设置为需要优化。

```python
class MyDense(layers.Layer): # 自定义网络层
	def __init__(self, inp_dim, outp_dim):
		super(MyDense, self).__init__()
		# 创建权值张量并添加到类管理列表中，设置为需要优化
		self.kernel = self.add_variable('w', [inp_dim, outp_dim],
		trainable=True)


net = MyDense(4,3) # 创建输入为 4，输出为 3 节点的自定义层 
net.variables,net.trainable_variables # 查看自定义层的参数列表
```

 - 通过修改为 self.kernel = self.add_variable('w', [inp_dim, outp_dim], trainable=False)，我们可以设置𝑾张量不需要被优化
 - 通过 tf.Variable 创建的类成员也会自动加入类参数列表

```python
# 通过 tf.Variable 创建的类成员也会自动加入类参数列表
self.kernel = tf.Variable(tf.random.normal([inp_dim, outp_dim]),
                trainable=False)
```

 - 自定义类的前向运算逻辑，对于这个例 子，只需要完成𝑶 = 𝑿@𝑾矩阵运算，并通过固定的 ReLU 激活函数即可

```python
def call(self, inputs, training=None): 
	# 实现自定义类的前向计算逻辑
	# X@W
	out = inputs @ self.kernel
	# 执行激活函数运算
	out = tf.nn.relu(out) return out
 
```

 - 自定义类的前向运算逻辑实现在 call(inputs, training=None)函数中，其中 inputs 代表输入，由用户在调用时传入;training 参数用于指定模型的状态:training 为 True 时执 行训练模式，training 为 False 时执行测试模式，默认参数为 None，即测试模式。由于全连 接层的训练模式和测试模式逻辑一致，此处不需要额外处理。对于部份测试模式和训练模 式不一致的网络层，需要根据 training 参数来设计需要执行的逻辑。

##  自定义网络

 - 自定义网络类可以和其他标准类一样，通过 Sequential 容器方便地封装成一个网络模型

```python
network = Sequential([MyDense(784, 256), # 使用自定义的层 MyDense(256, 128),
                 MyDense(128, 64),
                 MyDense(64, 32),
                 MyDense(32, 10)])
network.build(input_shape=(None, 28*28))
network.summary()
```

 - 创建自定义网络类，首先 创建类，并继承自 Model 基类，分别创建对应的网络层对象和前向运算逻辑

```python
class MyModel(keras.Model):
	# 自定义网络类，继承自 Model 基类 
	def __init__(self):
		super(MyModel, self).__init__() # 完成网络内需要的网络层的创建工作 
		self.fc1 = MyDense(28*28, 256) 
		self.fc2 = MyDense(256, 128) 
		self.fc3 = MyDense(128, 64)
		self.fc4 = MyDense(64, 32)
		self.fc5 = MyDense(32, 10)
	def call(self, inputs, training=None): # 自定义前向运算逻辑
		x = self.fc1(inputs)
		x = self.fc2(x)
		x = self.fc3(x)
		x = self.fc4(x)
		x = self.fc5(x)
		return x
```
#  常用模型
对于常用的网络模型，如 ResNet、VGG 等，不需要手动创建网络，可以直接从 keras.applications 子模块中通过一行代码即可创建并使用这些经典模型，同时还可以通过设 置 weights 参数加载预训练的网络参数。

##  加载模型

 - 以 ResNet50 网络模型为例，一般将 ResNet50 去除最后一层后的网络作为新任务的特 征提取子网络，即利用在 ImageNet 数据集上预训练好的网络参数初始化，并根据自定义任 务的类别追加一个对应数据类别数的全连接分类层或子网络，从而可以在预训练网络的基 础上快速、高效地学习新任务。
 - 首先利用 Keras 模型乐园加载 ImageNet 预训练好的 ResNet50 网络

```python
# 加载 ImageNet 预训练网络模型，并去掉最后一层
resnet = keras.applications.ResNet50(weights='imagenet',include_top=False) resnet.summary()
# 测试网络的输出
x = tf.random.normal([4,224,224,3])
out = resnet(x) # 获得子网络的输出
out.shape
```

 - 新建一个池化层(这里的池化层暂时 可以理解为高、宽维度下采样的功能)，将特征从[𝑏, 7,7,2048]降维到[𝑏, 2048]

```python
# 新建池化层
global_average_layer = layers.GlobalAveragePooling2D() # 利用上一层的输出作为本层的输入，测试其输出
x = tf.random.normal([4,7,7,2048])
# 池化层降维，形状由[4,7,7,2048]变为[4,1,1,2048],删减维度后变为[4,2048] 
out = global_average_layer(x)
print(out.shape)
```

 - 新建一个全连接层，并设置输出节点数为 100

```python
# 新建全连接层
fc = layers.Dense(100)
# 利用上一层的输出[4,2048]作为本层的输入，测试其输出
x = tf.random.normal([4,2048])
out = fc(x) # 输出层的输出为样本属于 100 类别的概率分布
print(out.shape)
```

#  测量工具
在网络的训练过程中，经常需要统计准确率、召回率等测量指标，除了可以通过手动 计算的方式获取这些统计数据外，Keras 提供了一些常用的测量工具，位于 keras.metrics 模 块中，专门用于统计训练过程中常用的指标数据。
##  新建测量器
在 keras.metrics 模块中，提供了较多的常用测量器类，如统计平均值的 Mean 类，统 计准确率的 Accuracy 类，统计余弦相似度的 CosineSimilarity 类等。下面我们以统计误差 值为例。在前向运算时，我们会得到每一个 Batch 的平均误差，但是我们希望统计每个 Step 的平均误差，因此选择使用 Mean 测量器。

```python
 # 新建平均测量器，适合 Loss 数据 
 loss_meter = metrics.Mean()
```
##  写入数据
通过测量器的 update_state 函数可以写入新的数据，测量器会根据自身逻辑记录并处理采样数据。例如，在每个 Step 结束时采集一次 loss 值。

```python
# 记录采样的数据，通过 float()函数将张量转换为普通数值
loss_meter.update_state(float(loss))
```
##   读取统计信息
在采样多次数据后，可以选择在需要的地方调用测量器的 result()函数，来获取统计值。例如，间隔性统计 loss 均值。

```python
 # 打印统计期间的平均 loss
print(step, 'loss:', loss_meter.result())
```
##  清除状态
由于测量器会统计所有历史记录的数据，因此在启动新一轮统计时，有必要清除历史 状态。通过 reset_states()即可实现清除状态功能。例如，在每次读取完平均误差后，清零统 计信息，以便下一轮统计的开始。

```python
if step % 100 == 0:
	# 打印统计的平均 loss
	print(step, 'loss:', loss_meter.result())
	loss_meter.reset_states() # 打印完后，清零测量器
```
##  准确率统计实战

```python
acc_meter = metrics.Accuracy() # 创建准确率测量器
# [b, 784] => [b, 10]，网络输出值
#Accuracy 类的 update_state 函数的参数为预测值和真实值，而不是当前 Batch 的准确率
out = network(x)
# [b, 10] => [b]，经过 argmax 后计算预测值 
pred = tf.argmax(out, axis=1)
pred = tf.cast(pred, dtype=tf.int32) # 根据预测值与真实值写入测量器
acc_meter.update_state(y, pred)
# 读取统计结果
print(step, 'Evaluate Acc:', acc_meter.result().numpy()) 
acc_meter.reset_states() # 清零测量器
```
#  可视化
TensorFlow 提供了一个专门的可视 化工具，叫做 TensorBoard，它通过 TensorFlow 将监控数据写入到文件系统，并利用 Web 后端监控对应的文件目录，从而可以允许用户从远程查看网络的监控数据。
##  

 - 通过 tf.summary.create_file_writer 创建监控对象类实例，并指定监控数据的写入目录

```python
 # 创建监控类，监控数据将写入 log_dir 目录
summary_writer = tf.summary.create_file_writer(log_dir)
```

 - 通过 tf.summary.scalar 函数记录监控数据，并指定时 间戳 step 参数

```python
with summary_writer.as_default(): # 写入环境
# 当前时间戳 step 上的数据为 loss，写入到名为 train-loss 数据库中
   tf.summary.scalar('train-loss', float(loss), step=step)
```

 - 通过 tf.summary.image 函数写入监控图片数据

```python
with summary_writer.as_default():# 写入环境
# 写入测试准确率
	tf.summary.scalar('test-acc', float(total_correct/total),
	step=step)
# 可视化测试用的图片，设置最多可视化 9 张图片
	tf.summary.image("val-onebyone-images:", val_images,
max_outputs=9, step=step)
```