---
title: Tensorflow与自编码器
date: 2020-12-06 21:39:03
categories: 
- 深度学习
tags:
- 深度学习框架
- python
- tensorflow
---
使用Tensorflow实现自编码器
<!-- more -->
#  自编码器原理
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201206163423731.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201206163436994.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201206163501759.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
#  Fashion MNIST 图片重建实战

 - 自编码器算法原理非常简单，实现方便，训练也较稳定，相对于 PCA 算法，神经网络 的强大表达能力可以学习到输入的高层抽象的隐藏特征向量𝒛，同时也能够基于𝒛重建出输入

##  Fashion MNIST 数据集

 - 利用keras.datasets.fashion_mnist.load_data()函数即可在线下载、管理和加载。

```python
# 加载Fashion MNIST图片数据集
(x_train, y_train), (x_test, y_test) = keras.datasets.fashion_mnist.load_dat a()
# 归一化
x_train, x_test = x_train.astype(np.float32) / 255., x_test.astype(np.float3 2) / 255.
# 只需要通过图片数据即可构建数据集对象，不需要标签
train_db = tf.data.Dataset.from_tensor_slices(x_train)
train_db = train_db.shuffle(batchsz * 5).batch(batchsz)
# 构建测试集对象
test_db = tf.data.Dataset.from_tensor_slices(x_test)
test_db = test_db.batch(batchsz)
```
##  编码器
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201206164019628.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

 - 利用 3 层的神经网络将长度为 784 的图片向量数据依次 降维到 256、128，最后降维到 h_dim 维度，每层使用 ReLU 激活函数，最后一层不使用激 活函数

```python
# 创建Encoders网络，实现在自编码器类的初始化函数中 
self.encoder = Sequential([
    layers.Dense(256, activation=tf.nn.relu),
	layers.Dense(128, activation=tf.nn.relu),
    layers.Dense(h_dim)
])
```
##  解码器

 - 基于隐藏向量 h_dim 依次升维到 128、256、784 长 度，除最后一层，激活函数使用 ReLU 函数。解码器的输出为 784 长度的向量，代表了打 平后的28 × 28大小图片，通过 Reshape 操作即可恢复为图片矩阵

```python
# 创建Decoders网络 
self.decoder = Sequential([
    layers.Dense(128, activation=tf.nn.relu),
    layers.Dense(256, activation=tf.nn.relu),
    layers.Dense(784)
])
```

##  自编码器
-  网络结构
```python
class AE(keras.Model):
# 自编码器模型类，包含了Encoder和Decoder2个子网络 
def __init__(self):
	super(AE, self).__init__() # 创建Encoders网络 
	self.encoder = Sequential([
	            layers.Dense(256, activation=tf.nn.relu),
	            layers.Dense(128, activation=tf.nn.relu),
	            layers.Dense(h_dim)
	])
	# 创建Decoders网络 
	self.decoder = Sequential([
	    layers.Dense(128, activation=tf.nn.relu),
	    layers.Dense(256, activation=tf.nn.relu),
	layers.Dense(784)
	])
```

 - 前向传播过程实现在 call 函数中，输入图片首先通过 encoder 子网络得到隐 藏向量 h，再通过 decoder 得到重建图片

```python
def call(self, inputs, training=None): # 前向传播函数
	# 编码获得隐藏向量h,[b, 784] => [b, 20] 
	h = self.encoder(inputs)
	# 解码获得重建图片，[b, 20] => [b, 784] 
	x_hat = self.decoder(h)
	return x_hat
```

##  网络训练

 - 创建自编码器实例和优化器，并设置合适的学习率

```python
# 创建网络对象
model = AE()
# 指定输入大小 
model.build(input_shape=(4, 784))
# 打印网络信息
model.summary()
# 创建优化器，并设置学习率
optimizer = optimizers.Adam(lr=lr)
```

 - 固定训练 100 个 Epoch，每次通过前向计算获得重建图片向量，并利用 tf.nn.sigmoid_cross_entropy_with_logits 损失函数计算重建图片与原始图片直接的误差，实 际上利用 MSE 误差函数也是可行的

```python
for epoch in range(100):

    for step, x in enumerate(train_db):

        #[b, 28, 28] => [b, 784]
        x = tf.reshape(x, [-1, 784])

        with tf.GradientTape() as tape:
            x_rec_logits = model(x)

            rec_loss = tf.losses.binary_crossentropy(x, x_rec_logits, from_logits=True)
            rec_loss = tf.reduce_mean(rec_loss)

        grads = tape.gradient(rec_loss, model.trainable_variables)
        optimizer.apply_gradients(zip(grads, model.trainable_variables))


        if step % 100 ==0:
            print(epoch, step, float(rec_loss))


        # evaluation
        x = next(iter(test_db))
        logits = model(tf.reshape(x, [-1, 784]))
        x_hat = tf.sigmoid(logits)
        # [b, 784] => [b, 28, 28]
        x_hat = tf.reshape(x_hat, [-1, 28, 28])

        # [b, 28, 28] => [2b, 28, 28]
        x_concat = tf.concat([x, x_hat], axis=0)
        x_concat = x_hat
        x_concat = x_concat.numpy() * 255.
        x_concat = x_concat.astype(np.uint8)
        save_images(x_concat, 'ae_images/rec_epoch_%d.png'%epoch)
```
##  图片重建

 - 为了测试图片重建效果，我们把数据集切分为训练集与测试集，其中测试集不参与训 练。我们从测试集中随机采样测试图片𝒙 ∈ 𝔻test，经过自编码器计算得到重建后的图片， 然后将真实图片与重建图片保存为图片阵列，并可视化，方便比对

```python
# 重建图片，从测试集采样一批图片 
x = next(iter(test_db))
logits = model(tf.reshape(x, [-1, 784])) # 打平并送入自编码器 
x_hat = tf.sigmoid(logits) 
# 将输出转换为像素值，使用sigmoid函数 # 恢复为 28x28,[b, 784] => [b, 28, 28]
x_hat = tf.reshape(x_hat, [-1, 28, 28])
# 输入的前50张+重建的前50张图片合并，[b, 28, 28] => [2b, 28, 28] 
x_concat = tf.concat([x[:50], x_hat[:50]], axis=0)
x_concat = x_concat.numpy() * 255. # 恢复为0~255范围
x_concat = x_concat.astype(np.uint8) # 转换为整型
save_images(x_concat, 'ae_images/rec_epoch_%d.png'%epoch) # 保存图片
```

 - save_images 函数负责将多张图片合并并保存为一张大图，这部分代码使用 PIL 图片库完成图片阵列逻辑

```python
def save_images(imgs, name):
	# 创建280x280大小图片阵列
	new_im = Image.new('L', (280, 280)) 
	index = 0
	for i in range(0, 280, 28): # 10 行图片阵列
		for j in range(0, 280, 28): # 10 列图片阵列
			im = imgs[index]
			im = Image.fromarray(im, mode='L') new_im.paste(im, (i, j)) # 写入对应位置
			index += 1 # 保存图片阵列
	new_im.save(name)
```
#  VAE 图片生成实战

 - 输入为 Fashion MNIST 图片向量，经过 3 个全连接层后得到隐向量𝐳的均值与方差，分别用两 个输出节点数为 20
   的全连接层表示，FC2 的 20 个输出节点表示 20 个特征分布的均值向量u，FC3 的 20 个输出节点表示 20
   个特征分布的取log后的方差向量。通过 Reparameterization Trick 采样获得长度为 20 的隐向量𝐳，并通过
   FC4 和 FC5 重建出样本图片。
 - VAE 作为生成模型，除了可以重建输入样本，还可以单独使用解码器生成样本。通过
   从先验分布𝑝(𝐳)中直接采样获得隐向量𝐳，经过解码后可以产生生成的样本。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201206170530570.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

##  VAE 模型

 - 创建 Encoder 和 Decoder 需要的网络层

```python
class VAE(keras.Model):

    def __init__(self):
        super(VAE, self).__init__()

        # Encoder
        self.fc1 = layers.Dense(128)
        self.fc2 = layers.Dense(z_dim) ## 均值输出
        self.fc3 = layers.Dense(z_dim)# 方差输出

        # Decoder# Decoder网络
        self.fc4 = layers.Dense(128)
        self.fc5 = layers.Dense(784)
```

 - Encoder 的输入先通过共享层 FC1，然后分别通过 FC2 与 FC3 网络，获得隐向量分布 的均值向量与方差的log向量值

```python
    def encoder(self, x):
		# 获得编码器的均值和方差
        h = tf.nn.relu(self.fc1(x))
        # 均值向量
        mu = self.fc2(h)
        # 方差的log向量
        log_var = self.fc3(h)

        return mu, log_var


```

 - Decoder 接受采样后的隐向量𝐳，并解码为图片输出

```python
    def decoder(self, z):
		# 根据隐藏变量z生成图片数据
        out = tf.nn.relu(self.fc4(z))
        out = self.fc5(out)
		# 返回图片数据，784向量
        return out
```

 - 在 VAE 的前向计算过程中，首先通过编码器获得输入的隐向量𝐳的分布，然后利用 Reparameterization Trick 实现的 reparameterize 函数采样获得隐向量𝐳，最后通过解码器即可 恢复重建的图片向量

```python
	def call(self, inputs, training=None): # 前向计算
	# 编码器[b, 784] => [b, z_dim], [b, z_dim] 
		mu, log_var = self.encoder(inputs)
		# 采样reparameterization trick
		z = self.reparameterize(mu, log_var)
		# 通过解码器生成
		x_hat = self.decoder(z)
		# 返回生成样本，及其均值与方差 
		return x_hat, mu, log_var
```

 - Reparameterize 函数接受均值与方差参数，并从正态分布𝒩(0, 𝐼)中采样获得𝜀，通过 z= 𝜇 + 𝜎 ⊙ 𝜀方式返回采样隐向量。

```python
	def reparameterize(self, mu, log_var):
		# reparameterize技巧，从正态分布采样epsion 
		eps = tf.random.normal(log_var.shape)
		# 计算标准差
		std = tf.exp(log_var)**0.5
		# reparameterize技巧
		z = mu + std * eps
		return z
```

##  网络训练

![- 网络固定训练 100 个 Epoch，每次从 VAE 模型中前向计算获得重建样本，通过交叉熵 损失函数计算重建误差项𝔼𝒛~𝑞\[𝑙𝑜𝑔𝑝𝜃(𝒙|𝒛)\]，根据公式(12-2)计算𝔻𝐾𝐿(𝑞 (𝒛|𝒙)||𝑝(𝒛))误差 项，并自动求导和更新整个网络模型。](https://img-blog.csdnimg.cn/20201206213226574.png)

```python
# 创建网络对象
model = VAE()
model.build(input_shape=(4, 784))
# 优化器
optimizer = tf.optimizers.Adam(lr)

for epoch in range(1000): # 训练100个Epoch

    for step, x in enumerate(train_db): # 遍历训练集
		# 打平，[b, 28, 28] => [b, 784]
        x = tf.reshape(x, [-1, 784])
		# 构建梯度记录器
        with tf.GradientTape() as tape:
        	# 前向计算
            x_rec_logits, mu, log_var = model(x)
			# 重建损失值计算
            rec_loss = tf.nn.sigmoid_cross_entropy_with_logits(labels=x, logits=x_rec_logits)
            rec_loss = tf.reduce_sum(rec_loss) / x.shape[0]

            # 计算KL散度 N(mu, var) VS N(0, 1)
			# 公式参考:https://stats.stackexchange.com/questions/7440/kl-divergence-between-two-univariate-gaussians
            kl_div = -0.5 * (log_var + 1 - mu**2 - tf.exp(log_var))
            kl_div = tf.reduce_sum(kl_div) / x.shape[0]
			# 合并误差项
            loss = rec_loss + 1. * kl_div
		# 自动求导
        grads = tape.gradient(loss, model.trainable_variables)
        # 自动更新
        optimizer.apply_gradients(zip(grads, model.trainable_variables))


        if step % 100 == 0:
            print(epoch, step, 'kl div:', float(kl_div), 'rec loss:', float(rec_loss))
```

##  图片生成

 - 图片生成只利用到解码器网络，首先从先验分布𝒩(0, 𝐼)中采样获得隐向量，再通过解 码器获得图片向量，最后 Reshape 为图片矩阵。

```python
# 测试生成效果，从正态分布随机采样z
z = tf.random.normal((batchsz, z_dim))
logits = model.decoder(z) # 仅通过解码器生成图片
x_hat = tf.sigmoid(logits) # 转换为像素范围
x_hat = tf.reshape(x_hat, [-1, 28, 28]).numpy() *255.
x_hat = x_hat.astype(np.uint8)
save_images(x_hat, 'vae_images/epoch_%d_sampled.png'%epoch) # 保存生成图片
# 重建图片，从测试集采样图片
x = next(iter(test_db))
logits, _, _ = model(tf.reshape(x, [-1, 784])) # 打平并送入自编码器 
x_hat = tf.sigmoid(logits) # 将输出转换为像素值
# 恢复为 28x28,[b, 784] => [b, 28, 28]
x_hat = tf.reshape(x_hat, [-1, 28, 28])
# 输入的前50张+重建的前50张图片合并，[b, 28, 28] => [2b, 28, 28] 
 x_concat = tf.concat([x[:50], x_hat[:50]], axis=0)
x_concat = x_concat.numpy() * 255. # 恢复为0~255范围
x_concat = x_concat.astype(np.uint8)
save_images(x_concat, 'vae_images/epoch_%d_rec.png'%epoch) # 保存重建图片
```
