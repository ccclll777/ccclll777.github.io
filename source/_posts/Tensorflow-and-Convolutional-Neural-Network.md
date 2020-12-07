---
title: Tensorflow与卷积神经网络
date: 2020-12-04 22:57:37
categories: 
- 深度学习
tags:
- 深度学习框架
- python
- tensorflow
---
使用Tensorflow实现卷积神经网络的相关算法
<!-- more -->
# 卷积层的实现
##  自定义权值

 - 在 TensorFlow 中，通过 tf.nn.conv2d 函数可以方便地实现 2D 卷积运算。tf.nn.conv2d 基于输入𝑿: [b.h,𝑤,𝑐𝑖𝑛] 和卷积核𝑾: [𝑘 𝑘 𝑐𝑖𝑛 𝑐𝑜𝑢𝑡] 进行卷积运算，得到输出𝑶 :[b,h′,𝑤′,𝑐𝑜𝑢𝑡] ，其中𝑐𝑖𝑛表示输入通道数，𝑐𝑜𝑢𝑡表示卷积核的数量，也是输出特征图的通道数。

```python
x = tf.random.normal([2,5,5,3]) # 模拟输入，3通道，高宽为5 
# 需要根据[k,k,cin,cout]格式创建W张量，4个3x3大小卷积核
w = tf.random.normal([3,3,3,4])
# 步长为1, padding为0,
out = tf.nn.conv2d(x,w,strides=1,padding=[[0,0],[0,0],[0,0],[0,0]])

```

 - 其中 padding 参数的设置格式为:padding=[[0,0],[上,下],[左,右],[0,0]]；
 - 上下左右各填充一个单位，则 padding 参数设置为   [[0,0],[1,1],[1,1],[0,0]]；

```python
x = tf.random.normal([2,5,5,3]) # 模拟输入，3通道，高宽为5 
# 需要根据[k,k,cin,cout]格式创建，4个3x3大小卷积核
w = tf.random.normal([3,3,3,4])
# 步长为1, padding为1,
out = tf.nn.conv2d(x,w,strides=1,padding=[[0,0],[1,1],[1,1],[0,0]])
```

 - 通过设置参数 padding='SAME'、strides=1 可以直接得到输入、输出同大小的 卷积层，其中 padding 的具体数量由 TensorFlow 自动计算并完成填充操作。

```python
x = tf.random.normal([2,5,5,3]) # 模拟输入，3通道，高宽为5 
w = tf.random.normal([3,3,3,4]) # 4 个 3x3 大小的卷积核
# 步长为,padding设置为输出、输入同大小
# 需要注意的是, padding=same只有在strides=1时才是同大小
out = tf.nn.conv2d(x,w,strides=1,padding='SAME')
```

 - 当𝑠> 时，设置padding='SAME'将使得输出高、宽将成1/s倍地减少

```python
x = tf.random.normal([2,5,5,3])
w = tf.random.normal([3,3,3,4])
# 高宽先padding成可以整除3的最小整数6，然后6按3倍减少，得到2x2 
out = tf.nn.conv2d(x,w,strides=3,padding='SAME')
```

 - 卷积神经网络层与全连接层一样，可以设置网络带偏置向量。tf.nn.conv2d 函数是没有 实现偏置向量计算的，添加偏置只需要手动累加偏置张量即可

```python
# 根据[cout]格式创建偏置向量 
b = tf.zeros([4])
# 在卷积输出上叠加偏置向量，它会自动broadcasting为[b,h',w',cout] 
out = out + b
```
##  卷积层类

 - 通过卷积层类 layers.Conv2D 可以不需要手动定义卷积核𝑾和偏置𝒃张量，直接调用类实例即可完成卷积层的前向计算，实现更加高层和快捷。
 - 在新建卷积层类时，只需要指定卷积核数量参数 filters，卷积核大小 kernel_size，步长 strides，填充 padding 等即可。如下创建了 4 个3 × 3大小的卷积核的卷积层，步长为 1， padding 方案为'SAME'：

```python
layer = layers.Conv2D(4,kernel_size=3,strides=1,padding='SAME')
```

 - 如果卷积核高宽不等，步长行列方向不等，此时需要将 kernel_size 参数设计为 tuple 格式(𝑘h 𝑘𝑤)，strides参数设计为(𝑠h 𝑠𝑤)。如下创建4个3× 大小的卷积核，竖直方向移 动步长𝑠h = 2，水平方向移动步长𝑠𝑤 = :

```python
layer = layers.Conv2D(4,kernel_size=(3,4),strides=(2,1),padding='SAME')
```

 - 创建完成后，通过调用实例(的__call__方法)即可完成前向计算：

```python
# 创建卷积层类
layer = layers.Conv2D(4,kernel_size=3,strides=1,padding='SAME') 
out = layer(x) # 前向计算
out.shape # 输出张量的shape
```

 - 在类 Conv2D 中，保存了卷积核张量𝑾和偏置𝒃，可以通过类成员 trainable_variables 直接返回𝑾和𝒃的列表

```python
 # 返回所有待优化张量列表 
 layer.trainable_variables
```


#  LeNet-5 实战

 - 它接受32 × 32大小的数字、字符图片，经过第一个卷积层得到 28 28 形状的张量，经过 一个向下采样层，张量尺寸缩小到 ，经过第二个卷积层，得到 形状 的张量，同样经过下采样层，张量尺寸缩小到 ，在进入全连接层之前，先将张量 打成 的张量，送入输出节点数分别为 120、84 的 2 个全连接层，得到 8 的张 量，最后通过 Gaussian connections 层。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201204222103242.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)


 - 在 LeNet-5 的基础上进行了少许调整，使得它更容易在现代深度学习框架上实 现。首先我们将输入𝑿形状由32 × 32调整为28 ×
   28，然后将 2 个下采样层实现为最大池化 层(降低特征图的高、宽，后续会介绍)，最后利用全连接层替换掉 Gaussian
   connections 层。下文统一称修改的网络也为 LeNet-5 网络：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201204222348393.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

```python
from tensorflow.keras import Sequential
network = Sequential([ # 网络容器 
layers.Conv2D(6,kernel_size=3,strides=1), # 第一个卷积层, 6个3x3卷积核 
layers.MaxPooling2D(pool_size=2,strides=2), # 高宽各减半的池化层
layers.ReLU(), # 激活函数
layers.Conv2D(16,kernel_size=3,strides=1), # 第二个卷积层, 16个3x3卷积核 
layers.MaxPooling2D(pool_size=2,strides=2), # 高宽各减半的池化层 
layers.ReLU(), # 激活函数
layers.Flatten(), # 打平层，方便全连接层处理
layers.Dense(120, activation='relu'), # 全连接层，120个节点
layers.Dense(84, activation='relu'), # 全连接层，84节点 
layers.Dense(10) # 全连接层，10个节点
])
# build一次网络模型，给输入X的形状，其中4为随意给的batchsz
network.build(input_shape=(4, 28, 28, 1)) 
# 统计网络信息
network.summary()
```

 - 在训练阶段，首先将数据集中 shape 为 28 28 的输入𝑿增加一个维度，调整 shape 为 28 28 ，送入模型进行前向计算，得到输出张量 output，shape 为 。我们新建交叉熵损失函数类(没错，损失函数也能使用类方式)用于处理分类任务，通过设定 from_logits=True 标志位将 softmax 激活函数实现在损失函数中，不需要手动添加损失函 数，提升数值计算稳定性。

```python
# 导入误差计算，优化器模块
from tensorflow.keras import losses, optimizers
# 创建损失函数的类，在实际计算时直接调用类实例即可
criteon = losses.CategoricalCrossentropy(from_logits=True)
# 构建梯度记录环境
with tf.GradientTape() as tape:
	# 插入通道维度，=>[b,28,28,1]
	x = tf.expand_dims(x,axis=3)
	# 前向计算，获得10类别的概率分布，[b, 784] => [b, 10]
	out = network(x)
	# 真实标签one-hot编码，[b] => [b, 10] y_onehot = tf.one_hot(y, depth=10) # 计算交叉熵损失函数，标量
	loss = criteon(y_onehot, out)
```

 - 获得损失值后，通过 TensorFlow 的梯度记录器 tf.GradientTape()来计算损失函数 loss 对网络 参数 network.trainable_variables 之间的梯度，并通过 optimizer 对象自动更新网络权 值参数

```python
# 自动计算梯度
grads = tape.gradient(loss, network.trainable_variables)
# 自动更新参数
optimizer.apply_gradients(zip(grads, network.trainable_variables))
```

 - 在测试阶段，由于不需要记录梯度信息，代码一般不需要写在 with tf.GradientTape() as tape 环境中。前向计算得到的输出经过 softmax 函数后，代表了 网络预测当前图片输入𝒙属于类别𝑖的概率𝑃(𝒙标签是𝑖|𝑥)，𝑖 ∈ 9 。通过 argmax 函数选取 概率最大的元素所在的索引，作为当前𝒙的预测类别，与真实标注𝑦比较，通过计算比较结 果中间 True 的数量并求和来统计预测正确的样本的个数，最后除以总样本的个数，得出网 络的测试准确度。

```python
# 记录预测正确的数量，总样本数量
correct, total = 0,0
for x,y in db_test: # 遍历所有训练集样本
	# 插入通道维度，=>[b,28,28,1]
	x = tf.expand_dims(x,axis=3)
	# 前向计算，获得10类别的预测分布，[b, 784] => [b, 10] out = network(x)
	# 真实的流程时先经过softmax，再argmax
	t32)))
	# 但是由于softmax不改变元素的大小相对关系，故省去 pred = tf.argmax(out, axis=-1)
	y = tf.cast(y, tf.int64)
	# 统计预测正确数量
	    correct += float(tf.reduce_sum(tf.cast(tf.equal(pred, y),tf.floa
	# 统计预测样本总数
	total += x.shape[0] # 计算准确率
print('test acc:', correct/total)
```
#  BatchNorm 层
参数标准化(Normalize)的手段， 并基于参数标准化设计了 Batch Nomalization(简写为 BatchNorm，或 BN)层 [6]。BN 层的 提出，使得网络的超参数的设定更加自由，比如更大的学习率、更随意的网络初始化等， 同时网络的收敛速度更快，性能也更好。BN 层提出后便广泛地应用在各种深度网络模型 上，卷积层、BN 层、ReLU 层、池化层一度成为网络模型的标配单元块，通过堆叠 Conv- BN-ReLU-Pooling 方式往往可以获得不错的模型性能。

##  BN 层实现

 - 在 TensorFlow 中，通过 layers.BatchNormalization()类可以非常方便地实现 BN 层
```python
 # 创建BN层 
 layer=layers.BatchNormalization()
```
 - 与全连接层、卷积层不同，BN 层的训练阶段和测试阶段的行为不同，需要通过设置 training 标志位来区分训练模式还是测试模式。
以 LeNet-5 的网络模型为例，在卷积层后添加 BN 层，代码如下:

```python
network = Sequential([ # 网络容器 layers.Conv2D(6,kernel_size=3,strides=1), # 插入BN层
	layers.BatchNormalization(),
	layers.MaxPooling2D(pool_size=2,strides=2), layers.ReLU(), layers.Conv2D(16,kernel_size=3,strides=1), # 插入BN层
	layers.BatchNormalization(),
	layers.MaxPooling2D(pool_size=2,strides=2),
	layers.ReLU(),
	layers.Flatten(),
	layers.Dense(120, activation='relu'), # 此处也可以插入BN层
	layers.Dense(84, activation='relu'), # 此处也可以插入BN层
	layers.Dense(10)
])
```
 - 在训练阶段，需要设置网络的参数 training=True 以区分 BN 层是训练还是测试模型

```python
with tf.GradientTape() as tape:
	# 插入通道维度
	x = tf.expand_dims(x,axis=3)
	# 前向计算，设置计算模式，[b, 784] => [b, 10] 
	out = network(x, training=True)
```

 - 在测试阶段，需要设置 training=False，避免 BN 层采用错误的行为

```python
for x,y in db_test: # 遍历测试集 # 插入通道维度
	x = tf.expand_dims(x,axis=3)
	# 前向计算，测试模式
	out = network(x, training=False)
```


# CIFAR10 与 VGG13 实战
```python
import  tensorflow as tf
from    tensorflow.keras import layers, optimizers, datasets, Sequential
import  os

os.environ['TF_CPP_MIN_LOG_LEVEL']='2'
tf.random.set_seed(2345)
# 先创建包含多网络层的列表
conv_layers = [ # 5 units of conv + max pooling
    # unit 1
# Conv-Conv-Pooling单元1
# 64个3x3卷积核, 输入输出同大小
    layers.Conv2D(64, kernel_size=[3, 3], padding="same", activation=tf.nn.relu),
    layers.Conv2D(64, kernel_size=[3, 3], padding="same", activation=tf.nn.relu),
    #高宽减半
    layers.MaxPool2D(pool_size=[2, 2], strides=2, padding='same'),

    # unit 2
    # Conv-Conv-Pooling单元2,输出通道提升至128，高宽大小减半
    layers.Conv2D(128, kernel_size=[3, 3], padding="same", activation=tf.nn.relu),
    layers.Conv2D(128, kernel_size=[3, 3], padding="same", activation=tf.nn.relu),
    layers.MaxPool2D(pool_size=[2, 2], strides=2, padding='same'),

    # unit 3
    # Conv-Conv-Pooling单元3,输出通道提升至256，高宽大小减半
    layers.Conv2D(256, kernel_size=[3, 3], padding="same", activation=tf.nn.relu),
    layers.Conv2D(256, kernel_size=[3, 3], padding="same", activation=tf.nn.relu),
    layers.MaxPool2D(pool_size=[2, 2], strides=2, padding='same'),

    # unit 4
    #Conv-Conv-Pooling单元4,输出通道提升至512，高宽大小减半
    layers.Conv2D(512, kernel_size=[3, 3], padding="same", activation=tf.nn.relu),
    layers.Conv2D(512, kernel_size=[3, 3], padding="same", activation=tf.nn.relu),
    layers.MaxPool2D(pool_size=[2, 2], strides=2, padding='same'),

    # unit 5
    #Conv-Conv-Pooling单元5,输出通道提升至512，高宽大小减半
    layers.Conv2D(512, kernel_size=[3, 3], padding="same", activation=tf.nn.relu),
    layers.Conv2D(512, kernel_size=[3, 3], padding="same", activation=tf.nn.relu),
    layers.MaxPool2D(pool_size=[2, 2], strides=2, padding='same')

]



def preprocess(x, y):
    # [0~1]
    x = 2*tf.cast(x, dtype=tf.float32) / 255.-1
    y = tf.cast(y, dtype=tf.int32)
    return x,y

# 在线下载，加载CIFAR10数据集
(x,y), (x_test, y_test) = datasets.cifar10.load_data()
# 删除y的一个维度，[b,1] => [b]
y = tf.squeeze(y, axis=1)
y_test = tf.squeeze(y_test, axis=1)
# 打印训练集和测试集的形状
print(x.shape, y.shape, x_test.shape, y_test.shape)

# 构建训练集对象，随机打乱，预处理，批量化
train_db = tf.data.Dataset.from_tensor_slices((x,y))
train_db = train_db.shuffle(1000).map(preprocess).batch(128)
# 构建测试集对象，预处理，批量化
test_db = tf.data.Dataset.from_tensor_slices((x_test,y_test))
test_db = test_db.map(preprocess).batch(64)
# 从训练集中采样一个Batch，并观察
sample = next(iter(train_db))
print('sample:', sample[0].shape, sample[1].shape,
      tf.reduce_min(sample[0]), tf.reduce_max(sample[0]))


def main():

    # [b, 32, 32, 3] => [b, 1, 1, 512]
    ## 利用前面创建的层列表构建网络容器
    conv_net = Sequential(conv_layers)
    ## 创建3层全连接层子网络
    fc_net = Sequential([
        layers.Dense(256, activation=tf.nn.relu),
        layers.Dense(128, activation=tf.nn.relu),
        layers.Dense(10, activation=None),
    ])
    # # build2个子网络，并打印网络参数信息
    conv_net.build(input_shape=[None, 32, 32, 3])
    fc_net.build(input_shape=[None, 512])
    conv_net.summary()
    fc_net.summary()
    optimizer = optimizers.Adam(lr=1e-4)

    # [1, 2] + [3, 4] => [1, 2, 3, 4]
    # 列表合并，合并2个子网络的参数
    variables = conv_net.trainable_variables + fc_net.trainable_variables

    for epoch in range(50):

        for step, (x,y) in enumerate(train_db):

            with tf.GradientTape() as tape:
                # [b, 32, 32, 3] => [b, 1, 1, 512]
                out = conv_net(x)
                # flatten, => [b, 512]
                out = tf.reshape(out, [-1, 512])
                # [b, 512] => [b, 10]
                logits = fc_net(out)
                # [b] => [b, 10]
                y_onehot = tf.one_hot(y, depth=10)
                # compute loss
                loss = tf.losses.categorical_crossentropy(y_onehot, logits, from_logits=True)
                loss = tf.reduce_mean(loss)
            # 对所有参数求梯度
            grads = tape.gradient(loss, variables)
            # 自动更新
            optimizer.apply_gradients(zip(grads, variables))

            if step %100 == 0:
                print(epoch, step, 'loss:', float(loss))



        total_num = 0
        total_correct = 0
        for x,y in test_db:

            out = conv_net(x)
            out = tf.reshape(out, [-1, 512])
            logits = fc_net(out)
            prob = tf.nn.softmax(logits, axis=1)
            pred = tf.argmax(prob, axis=1)
            pred = tf.cast(pred, dtype=tf.int32)

            correct = tf.cast(tf.equal(pred, y), dtype=tf.int32)
            correct = tf.reduce_sum(correct)

            total_num += x.shape[0]
            total_correct += int(correct)

        acc = total_correct / total_num
        print(epoch, 'acc:', acc)



if __name__ == '__main__':
    main()

```

