---
title: Tensorflow与循环神经网络
date: 2020-12-06 15:56:00
categories: 
- 深度学习
tags:
- 深度学习框架
- python
- tensorflow
---
                                                                    使用Tensorflow实现循环神经网络的相关算法
<!-- more -->
# 序列的表示方法
##  Embedding 层

 - 在神经网络中，单词的表示向量可以直接通过训练的方式得到，我们把单词的表示层 叫作 Embedding 层。
 - Embedding 层实现起来非常简单，构建一个 shape 为[𝑁vocab, 𝑛]的查询表对象 table，对于任意的单词编号𝑖，只需要查询到对应位置上的向量并返回即可:𝒗 = 𝑡𝑎𝑏𝑙𝑒[𝑖]
 - Embedding 层是可训练的，它可放置在神经网络之前，完成单词到向量的转换，得到的表 示向量可以继续通过神经网络完成后续任务，并计算误差L，采用梯度下降算法来实现端 到端(end-to-end)的训练。
 - 在 TensorFlow 中，可以通过 layers.Embedding(𝑁vocab,𝑛)来定义一个 Word Embedding 层，其中𝑁vocab参数指定词汇数量，𝑛指定单词向量的长度

```python
x = tf.range(10) # 生成 10 个单词的数字编码
x = tf.random.shuffle(x) # 打散
# 创建共 10 个单词，每个单词用长度为 4 的向量表示的层 
net = layers.Embedding(10, 4)
out = net(x) # 获取词向量

net.embeddings #查看 Embedding 层内部的查询表 table
```
##  预训练的词向量

 - 目前应用的比较广泛的预训练模型有 Word2Vec 和 GloVe 等。它们已经在海量语料库 训练得到了较好的词向量表示方法，并可以直接导出学习到的词向量表，方便迁移到其它任务。
 - 利用我们已经预训练好的模型参数去初始化 Embedding 层的查询表：

```python
# 从预训练模型中加载词向量表
embed_glove = load_embed('glove.6B.50d.txt') 
# 直接利用预训练的词向量表初始化 Embedding 层 
net.set_weights([embed_glove])
```

 - 预训练的词向量模型初始化的 Embedding 层可以设置为不参与训练:net.trainable =
   False，那么预训练的词向量就直接应用到此特定任务上;如果希望能够学到区别于预训 练词向量模型不同的表示方法，那么可以把
   Embedding 层包含进反向传播算法中去，利用 梯度下降来微调单词表示方法。

#  RNN 层使用方法

 - 在 TensorFlow 中，可以通过 layers.SimpleRNNCell 来完成𝜎(𝑾 𝒙𝑡 + 𝑾 𝑡−1 + 𝒃)计算。
 - 在 TensorFlow 中，RNN 表示通用意义上的循环神经网络，对于我们目前介绍的基础循环神经网络，它一般叫做 SimpleRNN。SimpleRNN 与 SimpleRNNCell 的 区别在于，带 Cell 的层仅仅是完成了一个时间戳的前向运算，不带 Cell 的层一般是基于 Cell 层实现的，它在内部已经完成了多个时间戳的循环运算，因此使用起来更为方便快捷。

##  SimpleRNNCell

 - 以某输入特征长度𝑛 = 4，Cell 状态向量特征长度h = 3为例，首先我们新建一个 SimpleRNNCell，不需要指定序列长度𝑠

```python
cell = layers.SimpleRNNCell(3) # 创建 RNN Cell，内存向量长度为 3 
cell.build(input_shape=(None,4)) # 输出特征长度 n=4
cell.trainable_variables # 打印 wxh, whh, b 张量
output:
[<tf.Variable 'kernel:0' shape=(4, 3) dtype=float32, numpy=...>,
 <tf.Variable 'recurrent_kernel:0' shape=(3, 3) dtype=float32, numpy=...>,
 <tf.Variable 'bias:0' shape=(3,) dtype=float32, numpy=array([0., 0., 0.],
dtype=float32)>]
```
SimpleRNNCell内部维护了3个张量，kernel变量即𝑾 张量，recurrent_kernel 变量即𝑾 张量，bias变量即偏置𝒃向量。但是RNN的Memory向量并不由SimpleRNNCell维护，需要用户自行初始化向量 𝟎并记录每个时间戳上的 ht.
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201205132144831.png)

 - 对于 SimpleRNNCell 来说，𝒐𝑡 = 𝑡，并没有经过额外的线性层转换，是同一个对象;[ 𝑡] 通过一个 List 包裹起来，这么设置是为了与 LSTM、GRU 等 RNN 变种格式统一。在循环 神经网络的初始化阶段，状态向量 𝟎一般初始化为全0向量.

```python
# 初始化状态向量，用列表包裹，统一格式
h0 = [tf.zeros([4, 64])]
x = tf.random.normal([4, 80, 100]) # 生成输入张量，4 个 80 单词的句子 
xt = x[:,0,:] # 所有句子的第 1 个单词
# 构建输入特征 n=100,序列长度 s=80,状态长度=64 的 Cell 
cell = layers.SimpleRNNCell(64)
out, h1 = cell(xt, h0) # 前向计算 
print(out.shape, h1[0].shape)
output：(4, 64) (4, 64)
```

 - 对于长度为𝑠的训练来说，需要循环通过 Cell 类𝑠次才算完成一次网络层的前向运算

```python
h = h0 # h保存每个时间戳上的状态向量列表 # 在序列长度的维度解开输入，得到 xt:[b,n]
for xt in tf.unstack(x, axis=1):
	out, h = cell(xt, h) # 前向计算,out 和 h 均被覆盖
# 最终输出可以聚合每个时间戳上的输出，也可以只取最后时间戳的输出 
out = out
```
##  多层 SimpleRNNCell 网络
通过在深度方向堆叠多个 Cell 类来实现深层卷积神经网络一样的效果，大大的提升网络的表达能力。

 - 利用 Cell 方式构建多层 RNN 网络。首先 新建两个 SimpleRNNCell 单元

```python
x = tf.random.normal([4,80,100]) 
xt = x[:,0,:] # 取第一个时间戳的输入 x0
# 构建 2 个 Cell,先 cell0,后 cell1，内存状态向量长度都为 64 
cell0 = layers.SimpleRNNCell(64)
cell1 = layers.SimpleRNNCell(64)
h0 = [tf.zeros([4,64])] # cell0的初始状态向量 
h1 = [tf.zeros([4,64])] # cell1的初始状态向量
```

 - 在时间轴上面循环计算多次来实现整个网络的前向运算，每个时间戳上的输入 xt 首先通过 第一层，得到输出 out0，再通过第二层，得到输出 out1

```python
for xt in tf.unstack(x, axis=1):
# xt作为输入，输出为out0
	out0, h0 = cell0(xt, h0)
# 上一个 cell 的输出 out0 作为本 cell 的输入
   out1, h1 = cell1(out0, h1)
```

 - 也可以先完成输入在第一层上所有时间戳的计算，并保存第一层在所有时间 戳上的输出列表，再计算第二层、第三层等的传播。

```python
# 保存上一层的所有时间戳上面的输出 
middle_sequences = []
# 计算第一层的所有时间戳上的输出，并保存 
for xt in tf.unstack(x, axis=1):
   out0, h0 = cell0(xt, h0)
   middle_sequences.append(out0)
# 计算第二层的所有时间戳上的输出
# 如果不是末层，需要保存所有时间戳上面的输出 
for xt in middle_sequences:
   out1, h1 = cell1(xt, h1)
```

 - 一般来说，最末层 Cell 的状态有可能保存了高层的全局语义特征，因此一般使用最末层的输出作为后续任务网络的输入。更特别地，每层最后一个时间戳上的状态输出包含了整个序列的全局信息，如果只希望选用 一个状态变量来完成后续任务，比如情感分类问题，一般选用最末层、最末时间戳的状态 输出最为合适。

##  SimpleRNN 层

 - 通过 SimpleRNN层高层接口可以非常方便地帮助我们实现自动进行循环神经网络内部的计算过程，比如每一层的 状态向量的初始化，以及每一层在时间轴上展开的运算

```python
layer = layers.SimpleRNN(64) # 创建状态向量长度为 64 的 SimpleRNN 层 
x = tf.random.normal([4, 80, 100])
out = layer(x) # 和普通卷积网络一样，一行代码即可获得输出
out.shape
```

 - 通过 SimpleRNN 可以仅需一行代码即可完成整个前向运算过程，它默认返回最 后一个时间戳上的输出。如果希望返回所有时间戳上的输出列表，可以设置 return_sequences=True 参数

```python
# 创建 RNN 层时，设置返回所有时间戳上的输出
layer = layers.SimpleRNN(64,return_sequences=True) 
out = layer(x) # 前向计算
out # 输出，自动进行了 concat 操作
```

 - 对于多层循环神经网络，我们可以通过堆叠多个 SimpleRNN 实现

```python
net = keras.Sequential([ # 构建 2 层 RNN 网络
# 除最末层外，都需要返回所有时间戳的输出，用作下一层的输入 
layers.SimpleRNN(64, return_sequences=True), 
layers.SimpleRNN(64),
])
out = net(x) # 前向计算
```
#  RNN 情感分类问题实战
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201205135047953.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
RNN 网络共两层，循环提取序列信号的语义特征，利用第2层RNN层的最后时间戳的状态向量 hs(2) 
作为句子的全局语义特征表示，送入全连接层构成的分类网络 3，得到样本𝒙为积极情感的概率P(𝒙为积极情感|𝒙) ∈ [0,1]

```python
#%%
import  os
import  tensorflow as tf
import  numpy as np
from    tensorflow import keras
from    tensorflow.keras import layers, losses, optimizers, Sequential


tf.random.set_seed(22)
np.random.seed(22)
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '2'
assert tf.__version__.startswith('2.')

batchsz = 128 # 批量大小
total_words = 10000 # 词汇表大小N_vocab
max_review_len = 80 # 句子最大长度s，大于的句子部分将截断，小于的将填充
embedding_len = 100 # 词向量特征长度f
# 加载IMDB数据集，此处的数据采用数字编码，一个数字代表一个单词
(x_train, y_train), (x_test, y_test) = keras.datasets.imdb.load_data(num_words=total_words)
print(x_train.shape, len(x_train[0]), y_train.shape)
print(x_test.shape, len(x_test[0]), y_test.shape)
#%%
x_train[0]
#%%
# 数字编码表
word_index = keras.datasets.imdb.get_word_index()
#
# 打印出编码表的单词和对应的数字
# for k,v in word_index.items():
#     print(k,v)
#%%
word_index = {k:(v+3) for k,v in word_index.items()}
word_index["<PAD>"] = 0  # 填充标志
word_index["<START>"] = 1  # 起始标志
word_index["<UNK>"] = 2  # unknown  未知单词的标志
word_index["<UNUSED>"] = 3
# 翻转编码表
reverse_word_index = dict([(value, key) for (key, value) in word_index.items()])
#对于一个数字编码的句子，通过如下函数转换为字符串数据
def decode_review(text):
    return ' '.join([reverse_word_index.get(i, '?') for i in text])

decode_review(x_train[8])

#%%

# x_train:[b, 80]
# x_test: [b, 80]
# 截断和填充句子，使得等长，此处长句子保留句子后面的部分，短句子在前面填充
x_train = keras.preprocessing.sequence.pad_sequences(x_train, maxlen=max_review_len)
x_test = keras.preprocessing.sequence.pad_sequences(x_test, maxlen=max_review_len)
# 构建数据集，打散，批量，并丢掉最后一个不够batchsz的batch
db_train = tf.data.Dataset.from_tensor_slices((x_train, y_train))
db_train = db_train.shuffle(1000).batch(batchsz, drop_remainder=True)
db_test = tf.data.Dataset.from_tensor_slices((x_test, y_test))
db_test = db_test.batch(batchsz, drop_remainder=True)
print('x_train shape:', x_train.shape, tf.reduce_max(y_train), tf.reduce_min(y_train))
print('x_test shape:', x_test.shape)

#%%

class MyRNN(keras.Model):
    # Cell方式构建多层网络
    def __init__(self, units):
        super(MyRNN, self).__init__()
        # [b, 64]，构建Cell初始化状态向量，重复使用
        self.state0 = [tf.zeros([batchsz, units])]
        self.state1 = [tf.zeros([batchsz, units])]
        # 词向量编码 [b, 80] => [b, 80, 100]
        self.embedding = layers.Embedding(total_words, embedding_len,
                                          input_length=max_review_len)
        # 构建2个Cell
        self.rnn_cell0 = layers.SimpleRNNCell(units, dropout=0.5)
        self.rnn_cell1 = layers.SimpleRNNCell(units, dropout=0.5)
        # 构建分类网络，用于将CELL的输出特征进行分类，2分类
        # [b, 80, 100] => [b, 64] => [b, 1]
        self.outlayer = Sequential([
        	layers.Dense(units),
        	layers.Dropout(rate=0.5),
        	layers.ReLU(),
        	layers.Dense(1)])

    def call(self, inputs, training=None):
        x = inputs # [b, 80]
        # embedding: [b, 80] => [b, 80, 100]
        # 获取词向量: [b, 80] => [b, 80, 100]
        x = self.embedding(x)
        # rnn cell compute,[b, 80, 100] => [b, 64]
        # 通过 2 个 RNN CELL,[b, 80, 100] => [b, 64]
        state0 = self.state0
        state1 = self.state1
        for word in tf.unstack(x, axis=1): # word: [b, 100] 
            out0, state0 = self.rnn_cell0(word, state0, training) 
            out1, state1 = self.rnn_cell1(out0, state1, training)
        # 末层最后一个输出作为分类网络的输入: [b, 64] => [b, 1]
        x = self.outlayer(out1, training)
        # p(y is pos|x)
        prob = tf.sigmoid(x)

        return prob

def main():
    units = 64 # RNN状态向量长度f
    epochs = 50 # 训练epochs

    model = MyRNN(units)
    # 装配
    model.compile(optimizer = optimizers.RMSprop(0.001),
                  loss = losses.BinaryCrossentropy(),
                  metrics=['accuracy'])
    # 训练和验证
    model.fit(db_train, epochs=epochs, validation_data=db_test)
    # 测试
    model.evaluate(db_test)


if __name__ == '__main__':
    main()

```
#  梯度弥散和梯度爆炸
##  梯度裁剪

 - 梯度爆炸可以通过梯度裁剪(Gradient Clipping)的方式在一定程度上的解决。梯度裁剪 与张量限幅非常类似，也是通过将梯度张量的数值或者范数限制在某个较小的区间内，从 而将远大于 1 的梯度值减少，避免出现梯度爆炸。
 - 直接对张量的数值进行限幅，使得张量𝑾的所有元素𝑤𝑖𝑗 ∈ [min, max]。在 TensorFlow中，可以通过 tf.clip_by_value()函数来实现。

```python
a=tf.random.uniform([2,2])
tf.clip_by_value(a,0.4,0.6) # 梯度值裁剪
```

 - 通过限制梯度张量𝑾的范数来实现梯度裁剪。比如对𝑾的二范数‖𝑾‖2约束在[0,max] 之间，如果‖𝑾‖2大于max值，则按照
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201205140232940.png)
方式将‖𝑾′‖2约束max内。可以通过 tf.clip_by_norm 函数方便的实现梯度张量𝑾裁剪

```python
a=tf.random.uniform([2,2]) * 5
# 按范数方式裁剪
b = tf.clip_by_norm(a, 5) 
# 裁剪前和裁剪后的张量范数 
tf.norm(a),tf.norm(b)
```

 - 神经网络的更新方向是由所有参数的梯度张量𝑾共同表示的，前两种方式只考虑单个 梯度张量的限幅，会出现网络更新方向发生变动的情况。如果能够考虑所有参数的梯 度𝑾的范数，实现等比例的缩放，那么就能既很好地限制网络的梯度值，同时不改变 网络的更新方向。这就是第三种梯度裁剪的方式:全局范数裁剪。在 TensorFlow 中， 可以通过 tf.clip_by_global_norm 函数快捷地缩放整体网络梯度𝑾的范数。
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201205140814287.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201205140822877.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
 

```python
w1=tf.random.normal([3,3]) # 创建梯度张量 1 
w2=tf.random.normal([3,3]) # 创建梯度张量 2
# 计算 global norm 
global_norm=tf.math.sqrt(tf.norm(w1)**2+tf.norm(w2)**2) 
# 根据 global norm 和 max norm=2 裁剪 
(ww1,ww2),global_norm=tf.clip_by_global_norm([w1,w2],2)
# 计算裁剪后的张量组的 global norm
global_norm2 = tf.math.sqrt(tf.norm(ww1)**2+tf.norm(ww2)**2)
# 打印裁剪前的全局范数和裁剪后的全局范数 
print(global_norm, global_norm2)
#tf.clip_by_global_norm 返回裁剪后的张量 List 和 global_norm 这两个对象，其中 global_norm 表示裁剪前的梯度总范数和
```

 - 在网络训练时，梯度裁剪一般在计算出梯度后，梯度更新之前进行

```python
with tf.GradientTape() as tape:
	logits = model(x) # 前向传播
	loss = criteon(y, logits) # 误差计算
	# 计算梯度值
	grads = tape.gradient(loss, model.trainable_variables)
	grads, _ = tf.clip_by_global_norm(grads, 25) 
	# 全局梯度裁剪 # 利用裁剪后的梯度张量更新参数
	optimizer.apply_gradients(zip(grads, model.trainable_variables))
```
##  梯度弥散
对于梯度弥散现象，可以通过增大学习率、减少网络深度、添加 Skip Connection 等一系列的措施抑制

#  LSTM 层使用方法

##  LSTMCell

 - LSTMCell 的用法和 SimpleRNNCell 基本一致，区别在于 LSTM 的状态变量 List 有两 个，即[ h𝑡,𝒄𝑡]，需要分别初始化，其中List第一个元素为h𝑡，第二个元素为𝒄𝑡。调用cell 完成前向运算时，返回两个元素，第一个元素为cell的输出，也就是 h𝑡，第二个元素为 cell的更新后的状态List:[ h𝑡,𝒄𝑡]。

```python
x = tf.random.normal([2,80,100]) 
xt = x[:,0,:] # 得到一个时间戳的输入
cell = layers.LSTMCell(64) # 创建 LSTM Cell # 初始化状态和输出 List,[h,c]
state = [tf.zeros([2,64]),tf.zeros([2,64])] 
out, state = cell(xt, state) # 前向计算
# 查看返回元素的 id
id(out),id(state[0]),id(state[1])
 
```

 - 通过在时间戳上展开循环运算，即可完成一次层的前向传播，写法与基础的 RNN 一 样

```python
# 在序列长度维度上解开，循环送入 LSTM Cell 单元 
for xt in tf.unstack(x, axis=1):
	# 前向计算
	out, state = cell(xt, state)
```
##  LSTM 层

 - 通过 layers.LSTM 层可以方便的一次完成整个序列的运算

```python
# 创建一层 LSTM 层，内存向量长度为 64
layer = layers.LSTM(64)
# 序列通过 LSTM 层，默认返回最后一个时间戳的输出 h 
out = layer(x)
```

 - 经过 LSTM 层前向传播后，默认只会返回最后一个时间戳的输出，如果需要返回每个时间 戳上面的输出，需要设置 return_sequences=True 标志

```python
# 创建 LSTM 层时，设置返回每个时间戳上的输出
layer = layers.LSTM(64, return_sequences=True)
# 前向计算，每个时间戳上的输出自动进行了 concat，拼成一个张量 
out = layer(x)
```

 - 对于多层神经网络，可以通过 Sequential 容器包裹多层 LSTM 层，并设置所有非末层 网络 return_sequences=True，这是因为非末层的 LSTM 层需要上一层在所有时间戳的输出作为输入

```python
net = keras.Sequential([
layers.LSTM(64, return_sequences=True), # 非末层需要返回所有时间戳输出
layers.LSTM(64)
])
# 一次通过网络模型，即可得到最末层、最后一个时间戳的输出 
out = net(x)
```


#   GRU层使用方法
门控循环网络(Gated Recurrent Unit，简称 GRU)是应用最广泛的 RNN 变种之一。GRU 把内部状态向量和输出向量合并，统一为状态向量 ，门控数量也减少到 2 个:复位门 (Reset Gate)和更新门(Update Gate).
##  GRU 使用方法

 - GRUCell 的使用，创建 GRU Cell 对象，并在时间轴上循环展开运算

```python
 # 初始化状态向量，GRU 只有一个
h = [tf.zeros([2,64])]
cell = layers.GRUCell(64) # 新建 GRU Cell，向量长度为 64
# 在时间戳维度上解开，循环通过 cell 
for xt in tf.unstack(x, axis=1):
	out, h = cell(xt, h)
# 输出形状 out.shape
out.shape
```

 - 通过 layers.GRU 类可以方便创建一层 GRU 网络层，通过 Sequential 容器可以堆叠多 层 GRU 层的网络

```python
net = keras.Sequential([
   layers.GRU(64, return_sequences=True),
   layers.GRU(64)
])
out = net(x)
```

#  LSTM/GRU 情感分类问题
##  LSTM 模型

 - Cell 方式。LSTM 网络的状态 List 共有两个，需要分别初始化各层的h和𝒄向量

```python
self.state0 = [tf.zeros([batchsz, units]),tf.zeros([batchsz, units])]
self.state1 = [tf.zeros([batchsz, units]),tf.zeros([batchsz, units])]
```

 - 将模型修改为 LSTMCell 模型

```python
self.rnn_cell0 = layers.LSTMCell(units, dropout=0.5)
self.rnn_cell1 = layers.LSTMCell(units, dropout=0.5)
```

 - 对于层方式，只需要修改网络模型一处

```python
# 构建 RNN，换成 LSTM 类即可 
self.rnn = keras.Sequential([
   layers.LSTM(units, dropout=0.5, return_sequences=True),
   layers.LSTM(units, dropout=0.5)
])
```
##  GRU模型

 -  Cell 方式。GRU 的状态 List 只有一个，和基础 RNN 一样，只需要修改创建 Cell 的类型

```python
 # 构建2个Cell
self.rnn_cell0 = layers.GRUCell(units, dropout=0.5) 
self.rnn_cell1 = layers.GRUCell(units, dropout=0.5)
```

 - 层方式，修改网络层类型

```python
# 构建RNN
self.rnn = keras.Sequential([
   layers.GRU(units, dropout=0.5, return_sequences=True),
layers.GRU(units, dropout=0.5)
])
```

#  预训练的词向量

 - 使用预训练的GloVe 词向量

```python
print('Indexing word vectors.')
embeddings_index = {} # 提取单词及其向量，保存在字典中
# 词向量模型文件存储路径
GLOVE_DIR = r'C:\Users\z390\Downloads\glove6b50dtxt'
with open(os.path.join(GLOVE_DIR, 'glove.6B.100d.txt'),encoding='utf-8') as f:
   for line in f:
       values = line.split()
	   word = values[0]
       coefs = np.asarray(values[1:], dtype='float32')
       embeddings_index[word] = coefs
print('Found %s word vectors.' % len(embeddings_index))
num_words = min(total_words, len(word_index))
embedding_matrix = np.zeros((num_words, embedding_len)) #词向量表 
for word, i in word_index.items():
	if i >= MAX_NUM_WORDS: 
		continue # 过滤掉其他词汇
embedding_vector = embeddings_index.get(word) # 从 GloVe 查询词向量 
	if embedding_vector is not None:
       # words not found in embedding index will be all-zeros.
	   embedding_matrix[i] = embedding_vector # 写入对应位置 
print(applied_vec_count, embedding_matrix.shape)
```

 - 获得了词汇表数据后，利用词汇表初始化 Embedding 层即可，并设置 Embedding 层 不参与梯度优化

```python
# 创建 Embedding 层
self.embedding = layers.Embedding(total_words, embedding_len,
                            input_length=max_review_len,
							trainable=False)#不参与梯度更新 
self.embedding.build(input_shape=(None, max_review_len))
# 利用 GloVe 模型初始化 Embedding 层 
self.embedding.set_weights([embedding_matrix])#初始化
```
