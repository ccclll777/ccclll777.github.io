---
title: Tensorflow与强化学习
date: 2020-12-08 20:46:47
categories: 
- 强化学习
tags:
- 深度学习框架
- python
- tensorflow
---
使用tensorflow框架实现强化学习算法，其中包括Policy Gradient ，A3C，DQN等算法
<!-- more -->
#  强化学习算法实例
##  平衡杆游戏
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020120810052028.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

 - 衡杆游戏系统包含了三个物体:滑轨、小车和杆。如图 ，小车可以自由在
   滑轨上移动，杆的一侧通过轴承固定在小车上。在初始状态，小车位于滑轨中央，杆竖直
   立在小车上，智能体通过控制小车的左右移动来控制杆的平衡，当杆与竖直方向的角度大
   于某个角度或者小车偏离滑轨中心位置一定距离后即视为游戏结束。游戏时间越长，游戏 给予的回报也就越多，智能体的操控水平也越高。
 - 为了简化环境状态的表示，我们这里直接取高层的环境特征向量𝑠作为智能体的输入，它一共包含了四个高层特征，分别为:小车位置、小车速度、杆角度和杆的速度。智能体的输出动作𝑎为向左移动或者向右移动，动作施加在平衡杆系统上会产生一个新的状态， 同时系统也会返回一个奖励值，这个奖励值可以简单的记为 1，即时长加一。在每个时间 戳𝑡上面，智能体通过观察环境状态𝑠𝑡而产生动作𝑎𝑡，环境接收动作后状态改变为𝑠𝑡+1，并返回奖励𝑟 。
##  Gym 平台
一般来说，在 Gym 环境中创建游戏并进行交互主要包含了 5 个步骤:
 - 创建游戏。通过 gym.make(name)即可创建指定名称 name 的游戏，并返回游戏对象 env。

 - 复位游戏状态。一般游戏环境都具有初始状态，通过调用 env.reset()即可复位游戏状 态，同时返回游戏的初始状态 observation。
 - 显示游戏画面。通过调用 env.render()即可显示每个时间戳的游戏画面，一般用做测 试。在训练时渲染画面会引入一定的计算代价，因此训练时可不显示画面。
 - 与游戏环境交互。通过 env.step(action)即可执行 action 动作，并返回新的状态 observation、当前奖励 reward、游戏是否结束标志 done 以及额外的信息载体 info。通 过循环此步骤即可持续与环境交互，直至游戏回合结束。
 - 销毁游戏。调用 env.close()即可。
 - 下面演示了一段平衡杆游戏 CartPole-v1 的交互代码，每次交互时在动作空间:{向左，向右}中随机采样一个动作，与环境进行交互，直至游戏结束。

```python
import gym # 导入 gym 游戏平台
env = gym.make("CartPole-v1") # 创建平衡杆游戏环境
observation = env.reset() # 复位游戏，回到初始状态 
for _ in range(1000): # 循环交互 1000 次
	env.render() # 显示当前时间戳的游戏画面
	action = env.action_space.sample() # 随机生成一个动作 
	# 与环境交互，返回新的状态，奖励，是否结束标志，其他信息 
	observation, reward, done, info = env.step(action) 
	if done:#游戏回合结束，复位状态
		observation = env.reset() 
env.close() # 销毁游戏环境
```
##  策略网络
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201208111441250.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

 - 将策略网络实现为一个 2 层的全连接网络，第一层将长度为 4 的向量转换为长度 为 128 的向量，第二层将 128 的向量转换为 2 的向量，即动作的概率分布

```python
class Policy(keras.Model):
    # 策略网络，生成动作的概率分布
    def __init__(self):
        super(Policy, self).__init__()
        self.data = [] # 存储轨迹
        # 输入为长度为4的向量，输出为左、右2个动作
        self.fc1 = layers.Dense(128, kernel_initializer='he_normal')
        self.fc2 = layers.Dense(2, kernel_initializer='he_normal')
        # 网络优化器
        self.optimizer = optimizers.Adam(lr=learning_rate)

    def call(self, inputs, training=None):
        # 状态输入s的shape为向量：[4]
        x = tf.nn.relu(self.fc1(inputs))
        x = tf.nn.softmax(self.fc2(x), axis=1)
        return x
```

 - 在交互时，我们将每个时间戳上的状态输入𝑠t ，动作分布输出𝑎t ，环境奖励𝑟t 和新状态 𝑠𝑡+1作为一个 4 元组 item 记录下来，用于策略网络的训练.

```python
    def put_data(self, item):
        # 记录r,log_P(a|s)z
        self.data.append(item)
```

 - 训练以及梯度更新

```python
    def train_net(self, tape):
        # 计算梯度并更新策略网络参数。tape为梯度记录器
        R = 0 # 终结状态的初始回报为0
        for r, log_prob in self.data[::-1]:#逆序取
            R = r + gamma * R # 计算每个时间戳上的回报
            # 每个时间戳都计算一次梯度
            # grad_R=-log_P*R*grad_theta
            loss = -log_prob * R
            with tape.stop_recording():
                # 优化策略网络
                grads = tape.gradient(loss, self.trainable_variables)
                # print(grads)  compute_gradients()返回的值作为输入参数对variable进行更新  防止梯度消失或者梯度爆炸
                self.optimizer.apply_gradients(zip(grads, self.trainable_variables))
        self.data = [] # 清空轨迹
```

 - 训练 400 个回合，在回合的开始，复位游戏状态，通过送入输入状态来采样动作，从而与环境进行交互，并记录每一个时间戳的信息，直至游戏回合结束

```python

def main():
    pi = Policy() # 创建策略网络
    pi(tf.random.normal((4,4)))
    pi.summary()
    score = 0.0 # 计分
    print_interval = 20 # 打印间隔
    returns = []

    for n_epi in range(400):
        s = env.reset() # 回到游戏初始状态，返回s0
        with tf.GradientTape(persistent=True) as tape:
            for t in range(501): # CartPole-v1 forced to terminates at 500 step.
                # 送入状态向量，获取策略
                s = tf.constant(s,dtype=tf.float32)
                # s: [4] => [1,4]  在第0个维度之前添加一个维度
                s = tf.expand_dims(s, axis=0)
                prob = pi(s) # 动作分布:[1,2]
                # 从类别分布中采样1个动作, shape: [1]
                a = tf.random.categorical(tf.math.log(prob), 1)[0]
                a = int(a) # Tensor转数字
                s_prime, r, done, info = env.step(a)
                # 记录动作a和动作产生的奖励r
                # prob shape:[1,2] 
                pi.put_data((r, tf.math.log(prob[0][a])))
                s = s_prime # 刷新状态
                score += r # 累积奖励

                if n_epi >1000:
                    env.render()
                    # im = Image.fromarray(s)
                    # im.save("res/%d.jpg" % info['frames'][0])

                if done:  # 当前episode终止
                    break
            # episode终止后，训练一次网络
            pi.train_net(tape)
        del tape

        if n_epi%print_interval==0 and n_epi!=0:
            returns.append(score/print_interval)
            print(f"# of episode :{n_epi}, avg score : {score/print_interval}")
            score = 0.0
    env.close() # 关闭环境

    plt.plot(np.arange(len(returns))*print_interval, returns)
    plt.plot(np.arange(len(returns))*print_interval, returns, 's')
    plt.xlabel('回合数')
    plt.ylabel('总回报')
    plt.savefig('reinforce-tf-cartpole.svg')

if __name__ == '__main__':
    main()
```

 - 完整代码

```python
import 	gym,os
import  numpy as np
import  matplotlib
from 	matplotlib import pyplot as plt
# Default parameters for plots
matplotlib.rcParams['font.size'] = 18
matplotlib.rcParams['figure.titlesize'] = 18
matplotlib.rcParams['figure.figsize'] = [9, 7]
matplotlib.rcParams['font.family'] = ['KaiTi']
matplotlib.rcParams['axes.unicode_minus']=False 

import 	tensorflow as tf
from    tensorflow import keras
from    tensorflow.keras import layers,optimizers,losses
from    PIL import Image
env = gym.make('CartPole-v1')  # 创建游戏环境
env.seed(2333)
tf.random.set_seed(2333)
np.random.seed(2333)
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '2'
assert tf.__version__.startswith('2.')

learning_rate = 0.0002
gamma         = 0.98

class Policy(keras.Model):
    # 策略网络，生成动作的概率分布
    def __init__(self):
        super(Policy, self).__init__()
        self.data = [] # 存储轨迹
        # 输入为长度为4的向量，输出为左、右2个动作
        self.fc1 = layers.Dense(128, kernel_initializer='he_normal')
        self.fc2 = layers.Dense(2, kernel_initializer='he_normal')
        # 网络优化器
        self.optimizer = optimizers.Adam(lr=learning_rate)

    def call(self, inputs, training=None):
        # 状态输入s的shape为向量：[4]
        x = tf.nn.relu(self.fc1(inputs))
        x = tf.nn.softmax(self.fc2(x), axis=1)
        return x

    def put_data(self, item):
        # 记录r,log_P(a|s)z
        self.data.append(item)

    def train_net(self, tape):
        # 计算梯度并更新策略网络参数。tape为梯度记录器
        R = 0 # 终结状态的初始回报为0
        for r, log_prob in self.data[::-1]:#逆序取
            R = r + gamma * R # 计算每个时间戳上的回报
            # 每个时间戳都计算一次梯度
            # grad_R=-log_P*R*grad_theta
            loss = -log_prob * R
            with tape.stop_recording():
                # 优化策略网络
                grads = tape.gradient(loss, self.trainable_variables)
                # print(grads)  compute_gradients()返回的值作为输入参数对variable进行更新  防止梯度消失或者梯度爆炸
                self.optimizer.apply_gradients(zip(grads, self.trainable_variables))
        self.data = [] # 清空轨迹

def main():
    pi = Policy() # 创建策略网络
    pi(tf.random.normal((4,4)))
    pi.summary()
    score = 0.0 # 计分
    print_interval = 20 # 打印间隔
    returns = []

    for n_epi in range(400):
        s = env.reset() # 回到游戏初始状态，返回s0
        with tf.GradientTape(persistent=True) as tape:
            for t in range(501): # CartPole-v1 forced to terminates at 500 step.
                # 送入状态向量，获取策略
                s = tf.constant(s,dtype=tf.float32)
                # s: [4] => [1,4]  在第0个维度之前添加一个维度
                s = tf.expand_dims(s, axis=0)
                prob = pi(s) # 动作分布:[1,2]
                # 从类别分布中采样1个动作, shape: [1]
                a = tf.random.categorical(tf.math.log(prob), 1)[0]
                a = int(a) # Tensor转数字
                s_prime, r, done, info = env.step(a)
                # 记录动作a和动作产生的奖励r
                # prob shape:[1,2]
                pi.put_data((r, tf.math.log(prob[0][a])))
                s = s_prime # 刷新状态
                score += r # 累积奖励

                if n_epi >1000:
                    env.render()
                    # im = Image.fromarray(s)
                    # im.save("res/%d.jpg" % info['frames'][0])

                if done:  # 当前episode终止
                    break
            # episode终止后，训练一次网络
            pi.train_net(tape)
        del tape

        if n_epi%print_interval==0 and n_epi!=0:
            returns.append(score/print_interval)
            print(f"# of episode :{n_epi}, avg score : {score/print_interval}")
            score = 0.0
    env.close() # 关闭环境

    plt.plot(np.arange(len(returns))*print_interval, returns)
    plt.plot(np.arange(len(returns))*print_interval, returns, 's')
    plt.xlabel('回合数')
    plt.ylabel('总回报')
    plt.savefig('reinforce-tf-cartpole.svg')

if __name__ == '__main__':
    main()
```

# 策略梯度方法（Policy Gradient ）
##  PPO 算法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201208142626810.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

 - **策略网络**：Actor 网络，策略网络的输入为状态𝑠𝑡，4 个输入节点，输出为动作𝑎𝑡 的概率分布𝜋𝜃(𝑎𝑡|𝑠𝑡)，采用 2 层的全连接层网络实现
。

```python
class Actor(keras.Model):
    def __init__(self):
        super(Actor, self).__init__()
        # 策略网络，也叫Actor网络，输出为概率分布pi(a|s)
        self.fc1 = layers.Dense(100, kernel_initializer='he_normal')
        self.fc2 = layers.Dense(2, kernel_initializer='he_normal')

    def call(self, inputs):
	    # 策略网络前向传播
        x = tf.nn.relu(self.fc1(inputs))
        x = self.fc2(x)
        # 输出2个动作的概率分布
        x = tf.nn.softmax(x, axis=1) # 转换成概率
        return x
```

 - **基准线𝑏值网络**： Critic 网络，或 V 值函数网络。网络的输入为状态𝑠𝑡，4 个输入 节点，输出为标量值𝑏，采用 2 层全连接层来估计𝑏。

```python
class Critic(keras.Model):
    def __init__(self):
        super(Critic, self).__init__()
        # 偏置b的估值网络，也叫Critic网络，输出为v(s)
        self.fc1 = layers.Dense(100, kernel_initializer='he_normal')
        self.fc2 = layers.Dense(1, kernel_initializer='he_normal')

    def call(self, inputs):
        x = tf.nn.relu(self.fc1(inputs))
        x = self.fc2(x)#输出基准线b的估计
        return x
```

 - 策略网络、值函数网络的创建工作，同时分别创建两个优化器，用于优化 策略网络和值函数网络的参数，我们创建在 PPO 算法主体类的初始化方法中

```python
class PPO():
    # PPO算法主体
    def __init__(self):
        super(PPO, self).__init__()
        self.actor = Actor() # 创建Actor网络
        self.critic = Critic() # 创建Critic网络
        self.buffer = [] # 数据缓冲池
        self.actor_optimizer = optimizers.Adam(1e-3) # Actor优化器
        self.critic_optimizer = optimizers.Adam(3e-3) # Critic优化器
```

 - **动作采样** 通过select_action 函数可以计算出当前状态的动作分布𝜋𝜃(𝑎𝑡|𝑠𝑡)，并根据概率随机采样动作，返回动作及其概率

```python
    def select_action(self, s):
        # 送入状态向量，获取策略: [4]
        s = tf.constant(s, dtype=tf.float32)
        # s: [4] => [1,4]   在第0个纬度之前插入一个纬度
        s = tf.expand_dims(s, axis=0)
        # 获取策略分布: [1, 2]
        prob = self.actor(s)
        # 从类别分布中采样1个动作, shape: [1]   tf.random.categorical 返回的是下标的列表
        a = tf.random.categorical(tf.math.log(prob), 1)[0]
        a = int(a)  # Tensor转数字
        return a, float(prob[0][a]) # 返回动作及其概率
```

 - **环境交互** 在主函数 main 中，与环境交互 500 个回合，每个回合通过 select_action 函 数采样策略，并保存进缓冲池，在间隔一段时间调用 agent.optimizer()函数优化策略。

```python
def main():
    agent = PPO()
    returns = [] # 统计总回报
    total = 0 # 一段时间内平均回报
    for i_epoch in range(500): # 训练回合数
        state = env.reset() # 复位环境
        for t in range(500): # 最多考虑500步
            # 通过最新策略与环境交互
            action, action_prob = agent.select_action(state)
            next_state, reward, done, _ = env.step(action)
            # 构建样本并存储  'state', 'action', 'a_log_prob 动作出现的概率', 'reward', 'next_state'
            trans = Transition(state, action, action_prob, reward, next_state)
            #存储状态
            agent.store_transition(trans)
            state = next_state # 刷新状态
            total += reward # 累积激励



            if done: # 合适的时间点训练网络
                if len(agent.buffer) >= batch_size:
                    # 交互一定轮次之后进行网络的训练
                    agent.optimize() # 训练网络
                break

        if i_epoch % 20 == 0: # 每20个回合统计一次平均回报
            returns.append(total/20)
            total = 0
            print(i_epoch, returns[-1])
```

 - **网络优化** 当缓冲池达到一定容量后，通过 optimizer()构建策略网络的误差和值网络的误差，优化网络的参数。首先将数据根据类别转换为 Tensor 类型，然后通过 MC 方法计算 累积回报𝑅(𝜏𝑡:𝑇 )。

> MC：蒙特卡罗法，蒙特卡罗法是一种不基于模型的强化问题求解方法。它可以 避免动态规划求解过于复杂，同时还可以不事先知道环境转化模 型，因此可以用于海量数据和复杂模型。但是它也有自己的缺点， 这就是它每次采样都需要一个完整的状态序列

```python
    def optimize(self):
        # 优化网络主函数
        # 从缓存中取出样本数据，转换成Tensor
        #状态
        state = tf.constant([t.state for t in self.buffer], dtype=tf.float32)
        #动作
        action = tf.constant([t.action for t in self.buffer], dtype=tf.int32)
        #转化成列向量
        action = tf.reshape(action,[-1,1])
        #奖励
        reward = [t.reward for t in self.buffer]
        #选择动作的概率
        old_action_log_prob = tf.constant([t.a_log_prob for t in self.buffer], dtype=tf.float32)
        old_action_log_prob = tf.reshape(old_action_log_prob, [-1,1])
        # 通过MC方法循环计算R(st)
        R = 0
        #存放累计回报的张量
        Rs = []
        #从最后一个开始循环
        for r in reward[::-1]:
            R = r + gamma * R
            Rs.insert(0, R)
        #构成张量
        Rs = tf.constant(Rs, dtype=tf.float32)
```

 - 对缓存池中的数据按 Batch Size 取出，迭代训练 10 遍。对于策略网络，根据 PPO2 算法的误差函数计算;对于值网络，通过均方差计算值网络的预测与𝑅(𝜏t，r )之间的距离，使得值网络的估计越来越准确。
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201208142526539.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201208142537282.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)



```python
    def optimize(self):
        # 优化网络主函数
        # 从缓存中取出样本数据，转换成Tensor
        #状态
        state = tf.constant([t.state for t in self.buffer], dtype=tf.float32)
        #动作
        action = tf.constant([t.action for t in self.buffer], dtype=tf.int32)
        #转化成列向量
        action = tf.reshape(action,[-1,1])
        #奖励
        reward = [t.reward for t in self.buffer]
        #选择动作的概率
        old_action_log_prob = tf.constant([t.a_log_prob for t in self.buffer], dtype=tf.float32)
        old_action_log_prob = tf.reshape(old_action_log_prob, [-1,1])
        # 通过MC方法循环计算R(st)
        R = 0
        #存放累计回报的张量
        Rs = []
        #从最后一个开始循环
        for r in reward[::-1]:
            R = r + gamma * R
            Rs.insert(0, R)
        #构成张量
        Rs = tf.constant(Rs, dtype=tf.float32)
        # 对缓冲池数据大致迭代10遍
        for _ in range(round(10*len(self.buffer)/batch_size)):
            # 随机从缓冲池采样batch size大小样本
            index = np.random.choice(np.arange(len(self.buffer)), batch_size, replace=False)
            # 构建梯度跟踪环境
            with tf.GradientTape() as tape1, tf.GradientTape() as tape2:
                # 取出R(st)，[b,1]  tf.gather 取出index对应的数据   然后扩展一个维度
                v_target = tf.expand_dims(tf.gather(Rs, index, axis=0), axis=1)
                # 计算v(s)预测值，也就是偏置b，我们后面会介绍为什么写成v  计算偏置b
                v = self.critic(tf.gather(state, index, axis=0))
                delta = v_target - v # 计算优势值
                advantage = tf.stop_gradient(delta) # 断开梯度连接 
                # 由于TF的gather_nd与pytorch的gather功能不一样，需要构造
                # gather_nd需要的坐标参数，indices:[b, 2]
                # pi_a = pi.gather(1, a) # pytorch只需要一行即可实现
                a = tf.gather(action, index, axis=0) # 取出batch的动作at
                # batch的动作分布pi(a|st)  每个动作出现的概率
                pi = self.actor(tf.gather(state, index, axis=0))
                #创建序列  扩展维度
                indices = tf.expand_dims(tf.range(a.shape[0]), axis=1)
                #与a进行拼接
                indices = tf.concat([indices, a], axis=1)
                pi_a = tf.gather_nd(pi, indices)  # 动作的概率值pi(at|st), [b]  #指定每次采样点的多维坐标来实现采样多个点的目的
                pi_a = tf.expand_dims(pi_a, axis=1)  # [b]=> [b,1] 
                # 重要性采样  不从原分布𝑝中进行采样，而通过另一个分布𝑞中进 行采样，只需要乘以𝑝(𝜏)/𝑞(𝜏)比率即可
                ratio = (pi_a / tf.gather(old_action_log_prob, index, axis=0))
                surr1 = ratio * advantage
                #实现上下限幅
                surr2 = tf.clip_by_value(ratio, 1 - epsilon, 1 + epsilon) * advantage
                # PPO误差函数
                policy_loss = -tf.reduce_mean(tf.minimum(surr1, surr2))
                # 对于偏置v来说，希望与MC估计的R(st)越接近越好
                value_loss = losses.MSE(v_target, v)
            # 优化策略网络
            grads = tape1.gradient(policy_loss, self.actor.trainable_variables)
            self.actor_optimizer.apply_gradients(zip(grads, self.actor.trainable_variables))
            # 优化偏置值网络
            grads = tape2.gradient(value_loss, self.critic.trainable_variables)
            self.critic_optimizer.apply_gradients(zip(grads, self.critic.trainable_variables))

        self.buffer = []  # 清空已训练数据
```

 - 整体代码

```python
import  matplotlib
from 	matplotlib import pyplot as plt
matplotlib.rcParams['font.size'] = 18
matplotlib.rcParams['figure.titlesize'] = 18
matplotlib.rcParams['figure.figsize'] = [9, 7]
matplotlib.rcParams['font.family'] = ['KaiTi']
matplotlib.rcParams['axes.unicode_minus']=False

plt.figure()

import  gym,os
import  numpy as np
import  tensorflow as tf
from    tensorflow import keras
from    tensorflow.keras import layers,optimizers,losses
from    collections import namedtuple
from    torch.utils.data import SubsetRandomSampler,BatchSampler

env = gym.make('CartPole-v1')  # 创建游戏环境
env.seed(2222)
tf.random.set_seed(2222)
np.random.seed(2222)
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '2'
assert tf.__version__.startswith('2.')



gamma = 0.98 # 激励衰减因子
epsilon = 0.2 # PPO误差超参数0.8~1.2
batch_size = 32 # batch size


# 创建游戏环境
env = gym.make('CartPole-v0').unwrapped
Transition = namedtuple('Transition', ['state', 'action', 'a_log_prob', 'reward', 'next_state'])


class Actor(keras.Model):
    def __init__(self):
        super(Actor, self).__init__()
        # 策略网络，也叫Actor网络，输出为概率分布pi(a|s)
        self.fc1 = layers.Dense(100, kernel_initializer='he_normal')
        self.fc2 = layers.Dense(2, kernel_initializer='he_normal')

    def call(self, inputs):
        x = tf.nn.relu(self.fc1(inputs))
        x = self.fc2(x)
        x = tf.nn.softmax(x, axis=1) # 转换成概率
        return x

class Critic(keras.Model):
    def __init__(self):
        super(Critic, self).__init__()
        # 偏置b的估值网络，也叫Critic网络，输出为v(s)
        self.fc1 = layers.Dense(100, kernel_initializer='he_normal')
        self.fc2 = layers.Dense(1, kernel_initializer='he_normal')

    def call(self, inputs):
        x = tf.nn.relu(self.fc1(inputs))
        x = self.fc2(x)
        return x




class PPO():
    # PPO算法主体
    def __init__(self):
        super(PPO, self).__init__()
        self.actor = Actor() # 创建Actor网络
        self.critic = Critic() # 创建Critic网络
        self.buffer = [] # 数据缓冲池
        self.actor_optimizer = optimizers.Adam(1e-3) # Actor优化器
        self.critic_optimizer = optimizers.Adam(3e-3) # Critic优化器

    def select_action(self, s):
        # 送入状态向量，获取策略: [4]
        s = tf.constant(s, dtype=tf.float32)
        # s: [4] => [1,4]   在第0个纬度之前插入一个纬度
        s = tf.expand_dims(s, axis=0)
        # 获取策略分布: [1, 2]
        prob = self.actor(s)
        # 从类别分布中采样1个动作, shape: [1]   tf.random.categorical 返回的是下标的列表
        a = tf.random.categorical(tf.math.log(prob), 1)[0]
        a = int(a)  # Tensor转数字
        return a, float(prob[0][a]) # 返回动作及其概率

    def get_value(self, s):
        # 送入状态向量，获取策略: [4]
        s = tf.constant(s, dtype=tf.float32)
        # s: [4] => [1,4]
        s = tf.expand_dims(s, axis=0)
        # 获取策略分布: [1, 2]
        v = self.critic(s)[0]
        return float(v) # 返回v(s)

    def store_transition(self, transition):
        # 存储采样数据
        self.buffer.append(transition)

    def optimize(self):
        # 优化网络主函数
        # 从缓存中取出样本数据，转换成Tensor
        #状态
        state = tf.constant([t.state for t in self.buffer], dtype=tf.float32)
        #动作
        action = tf.constant([t.action for t in self.buffer], dtype=tf.int32)
        #转化成列向量
        action = tf.reshape(action,[-1,1])
        #奖励
        reward = [t.reward for t in self.buffer]
        #选择动作的概率
        old_action_log_prob = tf.constant([t.a_log_prob for t in self.buffer], dtype=tf.float32)
        old_action_log_prob = tf.reshape(old_action_log_prob, [-1,1])
        # 通过MC方法循环计算R(st)
        R = 0
        #存放累计回报的张量
        Rs = []
        #从最后一个开始循环
        for r in reward[::-1]:
            R = r + gamma * R
            Rs.insert(0, R)
        #构成张量
        Rs = tf.constant(Rs, dtype=tf.float32)
        # 对缓冲池数据大致迭代10遍
        for _ in range(round(10*len(self.buffer)/batch_size)):
            # 随机从缓冲池采样batch size大小样本
            index = np.random.choice(np.arange(len(self.buffer)), batch_size, replace=False)
            # 构建梯度跟踪环境
            with tf.GradientTape() as tape1, tf.GradientTape() as tape2:
                # 取出R(st)，[b,1]  tf.gather 取出index对应的数据   然后扩展一个维度
                v_target = tf.expand_dims(tf.gather(Rs, index, axis=0), axis=1)
                # 计算v(s)预测值，也就是偏置b，我们后面会介绍为什么写成v  计算偏置b
                v = self.critic(tf.gather(state, index, axis=0))
                delta = v_target - v # 计算优势值
                advantage = tf.stop_gradient(delta) # 断开梯度连接 
                # 由于TF的gather_nd与pytorch的gather功能不一样，需要构造
                # gather_nd需要的坐标参数，indices:[b, 2]
                # pi_a = pi.gather(1, a) # pytorch只需要一行即可实现
                a = tf.gather(action, index, axis=0) # 取出batch的动作at
                # batch的动作分布pi(a|st)  每个动作出现的概率
                pi = self.actor(tf.gather(state, index, axis=0))
                #创建序列  扩展维度
                indices = tf.expand_dims(tf.range(a.shape[0]), axis=1)
                #与a进行拼接
                indices = tf.concat([indices, a], axis=1)
                pi_a = tf.gather_nd(pi, indices)  # 动作的概率值pi(at|st), [b]  #指定每次采样点的多维坐标来实现采样多个点的目的
                pi_a = tf.expand_dims(pi_a, axis=1)  # [b]=> [b,1] 
                # 重要性采样  不从原分布𝑝中进行采样，而通过另一个分布𝑞中进 行采样，只需要乘以𝑝(𝜏)/𝑞(𝜏)比率即可
                ratio = (pi_a / tf.gather(old_action_log_prob, index, axis=0))
                surr1 = ratio * advantage
                #实现上下限幅
                surr2 = tf.clip_by_value(ratio, 1 - epsilon, 1 + epsilon) * advantage
                # PPO误差函数
                policy_loss = -tf.reduce_mean(tf.minimum(surr1, surr2))
                # 对于偏置v来说，希望与MC估计的R(st)越接近越好
                value_loss = losses.MSE(v_target, v)
            # 优化策略网络
            grads = tape1.gradient(policy_loss, self.actor.trainable_variables)
            self.actor_optimizer.apply_gradients(zip(grads, self.actor.trainable_variables))
            # 优化偏置值网络
            grads = tape2.gradient(value_loss, self.critic.trainable_variables)
            self.critic_optimizer.apply_gradients(zip(grads, self.critic.trainable_variables))

        self.buffer = []  # 清空已训练数据


def main():
    agent = PPO()
    returns = [] # 统计总回报
    total = 0 # 一段时间内平均回报
    for i_epoch in range(500): # 训练回合数
        state = env.reset() # 复位环境
        for t in range(500): # 最多考虑500步
            # 通过最新策略与环境交互
            action, action_prob = agent.select_action(state)
            next_state, reward, done, _ = env.step(action)
            # 构建样本并存储  'state', 'action', 'a_log_prob 动作出现的概率', 'reward', 'next_state'
            trans = Transition(state, action, action_prob, reward, next_state)
            #存储状态
            agent.store_transition(trans)
            state = next_state # 刷新状态
            total += reward # 累积激励



            if done: # 合适的时间点训练网络
                if len(agent.buffer) >= batch_size:
                    # 交互一定轮次之后进行网络的训练
                    agent.optimize() # 训练网络
                break

        if i_epoch % 20 == 0: # 每20个回合统计一次平均回报
            returns.append(total/20)
            total = 0
            print(i_epoch, returns[-1])

    print(np.array(returns))
    plt.figure()
    plt.plot(np.arange(len(returns))*20, np.array(returns))
    plt.plot(np.arange(len(returns))*20, np.array(returns), 's')
    plt.xlabel('回合数')
    plt.ylabel('总回报')
    plt.savefig('ppo-tf-cartpole.svg')


if __name__ == '__main__':
    main()
    print("end")
```


#  值函数方法
策略梯度方法通过直接参数化策略网络，来优化得到更好的策略模型。在强化学习领 域，除了策略方法外，还有另外一类通过建模值函数而间接获得策略的方法，我们把它统 称为值函数方法。

##  值函数
状态值函数和状态-动作值函数，两者均表示在策略𝜋下的期望回报，轨迹起点定义不一样。

 - **状态值函数(State Value Function，简称 V 函数)**：从状态𝑠𝑡开始，在策略𝜋控 制下能获得的期望回报值:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201208145941156.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
 - **状态-动作值函数(State-Action Value Function，简称 Q 函数)**：从状态𝑠𝑡并执行 动作𝑎𝑡的双重设定下，在策略𝜋控制下能获得的期望回报值
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201208150024114.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
##  值函数的估计
 - **蒙特卡罗方法**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201208150610547.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
 - **时序差分方法**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201208150629891.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
##  策略改进
 - **ε-贪心法**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201208153008119.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
##  DQN 算法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201208153049582.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201208153104220.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
##  DQN 实战
 - **Q 网络**平衡杆游戏的状态是长度为 4 的向量，因此 Q 网络的输入设计为 4 个节点， 经过256 − 256 − 2的全连接层，得到输出节点数为 2 的 Q 函数估值的分布𝑄(𝑠, 𝑎)。

```python
class Qnet(keras.Model):
    def __init__(self):
        # 创建Q网络，输入为状态向量，输出为动作的Q值
        super(Qnet, self).__init__()
        self.fc1 = layers.Dense(256, kernel_initializer='he_normal')
        self.fc2 = layers.Dense(256, kernel_initializer='he_normal')
        self.fc3 = layers.Dense(2, kernel_initializer='he_normal')

    def call(self, x, training=None):
        x = tf.nn.relu(self.fc1(x))
        x = tf.nn.relu(self.fc2(x))
        x = self.fc3(x)
        return x
```

 - **经验回放池**在 DQN 算法中使用了经验回放池来减轻数据之间的强相关性，我们利用 ReplayBuffer 类中的 Deque 对象来实现缓存池的功能。在训练时，通过 put(transition)方法 将最新的(𝑠, 𝑎, 𝑟, 𝑠′)数据存入 Deque 对象，并通过 sample(n)方法从 Deque 对象中随机采样出 n 个(𝑠, 𝑎, 𝑟, 𝑠′)数据。

```python
class ReplayBuffer():
    # 经验回放池
    def __init__(self):
        # 双向队列
        self.buffer = collections.deque(maxlen=buffer_limit)
        #通过 put(transition)方法 将最新的(𝑠, 𝑎, 𝑟, 𝑠′)数据存入 Deque 对象
    def put(self, transition):
        self.buffer.append(transition)
    #通过 sample(n)方法从 Deque 对象中随机采样出 n 个(𝑠, 𝑎, 𝑟, 𝑠′)数据
    def sample(self, n):
        # 从回放池采样n个5元组
        mini_batch = random.sample(self.buffer, n)
        s_lst, a_lst, r_lst, s_prime_lst, done_mask_lst = [], [], [], [], []
        # 按类别进行整理
        for transition in mini_batch:
            s, a, r, s_prime, done_mask = transition
            s_lst.append(s)
            a_lst.append([a])
            r_lst.append([r])
            s_prime_lst.append(s_prime)
            done_mask_lst.append([done_mask])
        # 转换成Tensor
        return tf.constant(s_lst, dtype=tf.float32),\
                      tf.constant(a_lst, dtype=tf.int32), \
                      tf.constant(r_lst, dtype=tf.float32), \
                      tf.constant(s_prime_lst, dtype=tf.float32), \
                      tf.constant(done_mask_lst, dtype=tf.float32)


    def size(self):
        return len(self.buffer)
```

 - **策略改进** 这里实现了ε-贪心法。在采样动作时，有1 − ε的概率选择arg max 𝑄𝜋 (𝑠, 𝑎)，有ε的概率随机选择一个动作。

```python
    def sample_action(self, s, epsilon):
        # 送入状态向量，获取策略: [4]
        s = tf.constant(s, dtype=tf.float32)
        # s: [4] => [1,4]
        s = tf.expand_dims(s, axis=0)
        out = self(s)[0]
        coin = random.random()
        # 策略改进：e-贪心方式
        if coin < epsilon:
            # epsilon大的概率随机选取
            return random.randint(0, 1)
        else:  # 选择Q值最大的动作
            return int(tf.argmax(out))
```

 - **网络主流程**网络最多训练 10000 个回合，在回合开始时，首先复位游戏，得到初始状 态𝑠，并从当前 Q 网络中间采样一个动作，与环境进行交互，得到数据对(𝑠, 𝑎, 𝑟, 𝑠′)，并存 入经验回放池。如果当前经验回放池样本数量足够多，则采样一个 Batch 数据，根据 TD 误差优化 Q 网络的估值，直至游戏回合结束。

```python
    for n_epi in range(10000):  # 训练次数
        # epsilon概率也会8%到1%衰减，越到后面越使用Q值最大的动作
        epsilon = max(0.01, 0.08 - 0.01 * (n_epi / 200))
        s = env.reset()  # 复位环境
        for t in range(600):  # 一个回合最大时间戳
            # if n_epi>1000:
            #     env.render()
            # 根据当前Q网络提取策略，并改进策略
            a = q.sample_action(s, epsilon)
            # 使用改进的策略与环境交互
            s_prime, r, done, info = env.step(a)
            done_mask = 0.0 if done else 1.0  # 结束标志掩码
            # 保存5元组
            memory.put((s, a, r / 100.0, s_prime, done_mask))
            s = s_prime  # 刷新状态
            score += r  # 记录总回报
            if done:  # 回合结束
                break

        if memory.size() > 2000:  # 缓冲池只有大于2000就可以训练
            train(q, q_target, memory, optimizer)

        if n_epi % print_interval == 0 and n_epi != 0:
            for src, dest in zip(q.variables, q_target.variables):
                dest.assign(src)  # 影子网络权值来自Q
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/202012081649069.png)

 - **优化 Q 网络**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201208164928927.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

```python
def train(q, q_target, memory, optimizer):
    # 通过Q网络和影子网络来构造贝尔曼方程的误差，
    # 并只更新Q网络，影子网络的更新会滞后Q网络
    #Smooth L1 误差可以通过 Huber 误差类实现
    huber = losses.Huber()
    for i in range(10):  # 训练10次
        # 从缓冲池采样
        s, a, r, s_prime, done_mask = memory.sample(batch_size)
        with tf.GradientTape() as tape:
            # s: [b, 4]
            q_out = q(s)  # 得到Q(s,a)的分布
            # 由于TF的gather_nd与pytorch的gather功能不一样，需要构造
            # gather_nd需要的坐标参数，indices:[b, 2]
            # pi_a = pi.gather(1, a) # pytorch只需要一行即可实现
            indices = tf.expand_dims(tf.range(a.shape[0]), axis=1)
            indices = tf.concat([indices, a], axis=1)
            q_a = tf.gather_nd(q_out, indices) # 动作的概率值, [b]
            q_a = tf.expand_dims(q_a, axis=1) # [b]=> [b,1]
            # 得到Q(s',a)的最大值，它来自影子网络！ [b,4]=>[b,2]=>[b,1]
            max_q_prime = tf.reduce_max(q_target(s_prime),axis=1,keepdims=True)
            # 构造Q(s,a_t)的目标值，来自贝尔曼方程
            target = r + gamma * max_q_prime * done_mask
            # 计算Q(s,a_t)与目标值的误差
            loss = huber(q_a, target)
        # 更新网络，使得Q(s,a_t)估计符合贝尔曼方程
        grads = tape.gradient(loss, q.trainable_variables)
        # for p in grads:
        #     print(tf.norm(p))
        # print(grads)
        optimizer.apply_gradients(zip(grads, q.trainable_variables))
```

 - 完整代码

```python
import collections
import random
import gym,os
import  numpy as np
import  tensorflow as tf
from    tensorflow import keras
from    tensorflow.keras import layers,optimizers,losses

env = gym.make('CartPole-v1')  # 创建游戏环境
env.seed(1234)
tf.random.set_seed(1234)
np.random.seed(1234)
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '2'
assert tf.__version__.startswith('2.')

# Hyperparameters
learning_rate = 0.0002
gamma = 0.99
buffer_limit = 50000
batch_size = 32


class ReplayBuffer():
    # 经验回放池
    def __init__(self):
        # 双向队列
        self.buffer = collections.deque(maxlen=buffer_limit)
        #通过 put(transition)方法 将最新的(𝑠, 𝑎, 𝑟, 𝑠′)数据存入 Deque 对象
    def put(self, transition):
        self.buffer.append(transition)
    #通过 sample(n)方法从 Deque 对象中随机采样出 n 个(𝑠, 𝑎, 𝑟, 𝑠′)数据
    def sample(self, n):
        # 从回放池采样n个5元组
        mini_batch = random.sample(self.buffer, n)
        s_lst, a_lst, r_lst, s_prime_lst, done_mask_lst = [], [], [], [], []
        # 按类别进行整理
        for transition in mini_batch:
            s, a, r, s_prime, done_mask = transition
            s_lst.append(s)
            a_lst.append([a])
            r_lst.append([r])
            s_prime_lst.append(s_prime)
            done_mask_lst.append([done_mask])
        # 转换成Tensor
        return tf.constant(s_lst, dtype=tf.float32),\
                      tf.constant(a_lst, dtype=tf.int32), \
                      tf.constant(r_lst, dtype=tf.float32), \
                      tf.constant(s_prime_lst, dtype=tf.float32), \
                      tf.constant(done_mask_lst, dtype=tf.float32)


    def size(self):
        return len(self.buffer)


class Qnet(keras.Model):
    def __init__(self):
        # 创建Q网络，输入为状态向量，输出为动作的Q值
        super(Qnet, self).__init__()
        self.fc1 = layers.Dense(256, kernel_initializer='he_normal')
        self.fc2 = layers.Dense(256, kernel_initializer='he_normal')
        self.fc3 = layers.Dense(2, kernel_initializer='he_normal')

    def call(self, x, training=None):
        x = tf.nn.relu(self.fc1(x))
        x = tf.nn.relu(self.fc2(x))
        x = self.fc3(x)
        return x

    def sample_action(self, s, epsilon):
        # 送入状态向量，获取策略: [4]
        s = tf.constant(s, dtype=tf.float32)
        # s: [4] => [1,4]
        s = tf.expand_dims(s, axis=0)
        out = self(s)[0]
        coin = random.random()
        # 策略改进：e-贪心方式
        if coin < epsilon:
            # epsilon大的概率随机选取
            return random.randint(0, 1)
        else:  # 选择Q值最大的动作
            return int(tf.argmax(out))


def train(q, q_target, memory, optimizer):
    # 通过Q网络和影子网络来构造贝尔曼方程的误差，
    # 并只更新Q网络，影子网络的更新会滞后Q网络
    #Smooth L1 误差可以通过 Huber 误差类实现
    huber = losses.Huber()
    for i in range(10):  # 训练10次
        # 从缓冲池采样
        s, a, r, s_prime, done_mask = memory.sample(batch_size)
        with tf.GradientTape() as tape:
            # s: [b, 4]
            q_out = q(s)  # 得到Q(s,a)的分布
            # 由于TF的gather_nd与pytorch的gather功能不一样，需要构造
            # gather_nd需要的坐标参数，indices:[b, 2]
            # pi_a = pi.gather(1, a) # pytorch只需要一行即可实现
            indices = tf.expand_dims(tf.range(a.shape[0]), axis=1)
            indices = tf.concat([indices, a], axis=1)
            q_a = tf.gather_nd(q_out, indices) # 动作的概率值, [b]
            q_a = tf.expand_dims(q_a, axis=1) # [b]=> [b,1]
            # 得到Q(s',a)的最大值，它来自影子网络！ [b,4]=>[b,2]=>[b,1]
            max_q_prime = tf.reduce_max(q_target(s_prime),axis=1,keepdims=True)
            # 构造Q(s,a_t)的目标值，来自贝尔曼方程
            target = r + gamma * max_q_prime * done_mask
            # 计算Q(s,a_t)与目标值的误差
            loss = huber(q_a, target)
        # 更新网络，使得Q(s,a_t)估计符合贝尔曼方程
        grads = tape.gradient(loss, q.trainable_variables)
        # for p in grads:
        #     print(tf.norm(p))
        # print(grads)
        optimizer.apply_gradients(zip(grads, q.trainable_variables))


def main():
    env = gym.make('CartPole-v1')  # 创建环境
    q = Qnet()  # 创建Q网络
    q_target = Qnet()  # 创建影子网络
    q.build(input_shape=(2,4))
    q_target.build(input_shape=(2,4))
    for src, dest in zip(q.variables, q_target.variables):
        dest.assign(src) # 影子网络权值来自Q
    memory = ReplayBuffer()  # 创建回放池

    print_interval = 20
    score = 0.0
    optimizer = optimizers.Adam(lr=learning_rate)

    for n_epi in range(10000):  # 训练次数
        # epsilon概率也会8%到1%衰减，越到后面越使用Q值最大的动作
        epsilon = max(0.01, 0.08 - 0.01 * (n_epi / 200))
        s = env.reset()  # 复位环境
        for t in range(600):  # 一个回合最大时间戳
            # if n_epi>1000:
            #     env.render()
            # 根据当前Q网络提取策略，并改进策略
            a = q.sample_action(s, epsilon)
            # 使用改进的策略与环境交互
            s_prime, r, done, info = env.step(a)
            done_mask = 0.0 if done else 1.0  # 结束标志掩码
            # 保存5元组
            memory.put((s, a, r / 100.0, s_prime, done_mask))
            s = s_prime  # 刷新状态
            score += r  # 记录总回报
            if done:  # 回合结束
                break

        if memory.size() > 2000:  # 缓冲池只有大于2000就可以训练
            train(q, q_target, memory, optimizer)

        if n_epi % print_interval == 0 and n_epi != 0:
            for src, dest in zip(q.variables, q_target.variables):
                dest.assign(src)  # 影子网络权值来自Q
            print("# of episode :{}, avg score : {:.1f}, buffer size : {}, " \
                  "epsilon : {:.1f}%" \
                  .format(n_epi, score / print_interval, memory.size(), epsilon * 100))
            score = 0.0
    env.close()


if __name__ == '__main__':
    main()
```
#  Actor-Critic 方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201208193539146.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
##  Advantage AC 算法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201208193622351.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
##  A3C 算法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201208193638743.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

 - A3C 算法全称为 Asynchronous Advantage Actor-Critic 算法，是 DeepMind 基于Advantage Actor-Critic 算法提出来的异步版本 [8]，将 Actor-Critic 网络部署在多个线程中
   同时进行训练，并通过全局网络来同步参数。这种异步训练的模式大大提升了训练效率，训练速度更快，并且算法性能也更好。
 - 如图 ，算法会新建一个全局网络 Global Network 和 M 个 Worker 线程， Global Network 包含了Actor 和 Critic 网络，每个线程均新建一个交互环境和 Actor 和 Critic 网络。初始化阶段 Global Network 随机初始化参数𝜃 和𝜙 ，Worker 中的 Actor-Critic 网络从 Global Network中同步拉取参数来初始化网络。在训练时，Worker 中的 Actor-Critic 网络首先从 Global Network拉取最新参数，然后在最新策略𝜋𝜃(𝑎𝑡|𝑠𝑡)才采样动作与私有环 境进行交互，并根据 Advantage Actor-Critic 算法方法计算参数𝜃 和𝜙的梯度信息。完成梯 度计算后，各个 Worker 将梯度信息提交到 Global Network 中，利用 Global Network 的优化 器完成 Global Network的网络参数更新。在算法测试阶段，只使用Global Network 与环境交互即可。

##  A3C 实战

 - 异步的 A3C 算法。和普通的 Advantage AC 算法一样，需要创建 ActorCritic 网络大类，它包含了一个 Actor 子网络和一个 Critic 子网络，有时 Actor 和 Critic 会共享前面网络数层，减少网络的参数量。平衡杆游戏比较简单，我们使用一个 2 层 全连接网络来参数化 Actor 网络，使用另一个 2 层全连接网络来参数化 Critic 网络。

```python
class ActorCritic(keras.Model):
    # Actor-Critic模型
    def __init__(self, state_size, action_size):
        super(ActorCritic, self).__init__()
        self.state_size = state_size # 状态向量长度
        self.action_size = action_size # 动作数量
        # 策略网络Actor
        self.dense1 = layers.Dense(128, activation='relu')
        self.policy_logits = layers.Dense(action_size)

        # V网络Critic
        self.dense2 = layers.Dense(128, activation='relu')
        self.values = layers.Dense(1)
```

 - Actor-Critic 的前向传播过程分别计算策略分布𝜋𝜃(𝑎𝑡|𝑠𝑡)和 V 函数估计𝑉𝜋(𝑠𝑡)

```python
    def call(self, inputs):
        # 获得策略分布Pi(a|s)
        x = self.dense1(inputs)
        logits = self.policy_logits(x)
        # 获得v(s)
        v = self.dense2(inputs)
        values = self.values(v)
        return logits, values
```

 - **Worker 线程类** 在 Worker 线程中，实现和 Advantage AC 算法一样的计算流程，只是 计算产生的参数𝜃 和𝜙的梯度信息并不直接用于更新 Worker 的 Actor-Critic 网络，而是提 交到 Global Network 更新。具体地，在 Worker 类初始化阶段，获得 Global Network 传入的 server 对象和 opt 对象，分别代表了 Global Network 模型和优化器;并创建私有的 ActorCritic 网络类 client 和交互环境 env。

```python
class Worker(threading.Thread): 
    def __init__(self,  server, opt, result_queue, idx):
        super(Worker, self).__init__()
        self.result_queue = result_queue # 共享队列
        self.server = server # 中央模型
        self.opt = opt # 中央优化器
        self.client = ActorCritic(4, 2) # 线程私有网络
        self.worker_idx = idx # 线程id
        self.env = gym.make('CartPole-v1').unwrapped
        self.ep_loss = 0.0
```

 - 在线程运行阶段，每个线程最多与环境交互 400 个回合，在回合开始，利用 client 网 络采样动作与环境进行交互，并保存至 Memory 对象。在回合结束，训练 Actor 网络和 Critic 网络，得到参数𝜃 和𝜙的梯度信息，调用 Global Network 的 opt 优化器对象更新 Global Network。

```python
    def run(self): 
        mem = Memory() # 每个worker自己维护一个memory
        for epi_counter in range(500): # 未达到最大回合数
            current_state = self.env.reset() # 复位client游戏状态
            mem.clear()
            ep_reward = 0.
            ep_steps = 0  
            done = False
            while not done:
                # 获得Pi(a|s),未经softmax
                logits, _ = self.client(tf.constant(current_state[None, :],
                                         dtype=tf.float32))
                probs = tf.nn.softmax(logits)
                # 随机采样动作
                action = np.random.choice(2, p=probs.numpy()[0])
                new_state, reward, done, _ = self.env.step(action) # 交互 
                ep_reward += reward # 累加奖励
                mem.store(current_state, action, reward) # 记录
                ep_steps += 1 # 计算回合步数
                current_state = new_state # 刷新状态 

                if ep_steps >= 500 or done: # 最长步数500
                    # 计算当前client上的误差
                    with tf.GradientTape() as tape:
                        total_loss = self.compute_loss(done, new_state, mem) 
                    # 计算误差
                    grads = tape.gradient(total_loss, self.client.trainable_weights)
                    # 梯度提交到server，在server上更新梯度
                    self.opt.apply_gradients(zip(grads,
                                                 self.server.trainable_weights))
                    # 从server拉取最新的梯度
                    self.client.set_weights(self.server.get_weights())
                    mem.clear() # 清空Memory 
                    # 统计此回合回报
                    self.result_queue.put(ep_reward)
                    print(self.worker_idx, ep_reward)
                    break
        self.result_queue.put(None) # 结束线程
```

 - **Actor-Critic 误差计算**
![ ](https://img-blog.csdnimg.cn/2020120819533547.png)

```python
    def compute_loss(self,
                     done,
                     new_state,
                     memory,
                     gamma=0.99):
        if done:
            reward_sum = 0. # 终止状态的v(终止)=0
        else:
            reward_sum = self.client(tf.constant(new_state[None, :],
                                     dtype=tf.float32))[-1].numpy()[0]
        # 统计折扣回报
        discounted_rewards = []
        for reward in memory.rewards[::-1]:  # reverse buffer r
            reward_sum = reward + gamma * reward_sum
            discounted_rewards.append(reward_sum)
        discounted_rewards.reverse()
        # 获取状态的Pi(a|s)和v(s)
        logits, values = self.client(tf.constant(np.vstack(memory.states),
                                 dtype=tf.float32))
        # 计算advantage = R() - v(s)
        advantage = tf.constant(np.array(discounted_rewards)[:, None],
                                         dtype=tf.float32) - values
        # Critic网络损失
        value_loss = advantage ** 2
        # 策略损失
        policy = tf.nn.softmax(logits)
        policy_loss = tf.nn.sparse_softmax_cross_entropy_with_logits(
                        labels=memory.actions, logits=logits)
        # 计算策略网络损失时，并不会计算V网络
        policy_loss = policy_loss * tf.stop_gradient(advantage)
        # Entropy Bonus  labels标签值（真实值）logits模型的输出
        entropy = tf.nn.softmax_cross_entropy_with_logits(labels=policy,
                                                          logits=logits)
        policy_loss = policy_loss - 0.01 * entropy
        # 聚合各个误差
        total_loss = tf.reduce_mean((0.5 * value_loss + policy_loss))
        return total_loss
```

 - **智能体**负责整个 A3C 算法的训练。在智能体类初始化阶段，新建 Global Network 全局网络对象 server 和它的优化器对象 opt。
在训练开始时，创建各个 Worker 线程对象，并启动各个线程对象与环境交互，每个 Worker 对象在交互时均会从 Global Network 中拉取最新的网络参数，并利用最新策略与环 境交互，计算各自损失函数，最后提交梯度信息给 Global Network，调用 opt 对象完成 Global Network 的优化更新。
```python
class Agent:
    # 智能体，包含了中央参数网络server
    def __init__(self):
        # server优化器，client不需要，直接从server拉取参数
        self.opt = optimizers.Adam(1e-3)
        # 中央模型，类似于参数服务器
        self.server = ActorCritic(4, 2) # 状态向量，动作数量
        self.server(tf.random.normal((2, 4)))
    def train(self):
        res_queue = Queue() # 共享队列
        # 创建各个交互环境
        workers = [Worker(self.server, self.opt, res_queue, i)
                   for i in range(multiprocessing.cpu_count())]
        for i, worker in enumerate(workers):
            print("Starting worker {}".format(i))
            worker.start()
        # 统计并绘制总回报曲线
        returns = []
        while True:
            reward = res_queue.get()
            if reward is not None:
                returns.append(reward)
            else: # 结束标志
                break
        [w.join() for w in workers] # 等待线程退出 

        print(returns)
        plt.figure()
        plt.plot(np.arange(len(returns)), returns)
        # plt.plot(np.arange(len(moving_average_rewards)), np.array(moving_average_rewards), 's')
        plt.xlabel('回合数')
        plt.ylabel('总回报')
        plt.savefig('a3c-tf-cartpole.svg')
```
-  完整代码

```python
import  matplotlib
from    matplotlib import pyplot as plt
matplotlib.rcParams['font.size'] = 18
matplotlib.rcParams['figure.titlesize'] = 18
matplotlib.rcParams['figure.figsize'] = [9, 7]
matplotlib.rcParams['font.family'] = ['KaiTi']
matplotlib.rcParams['axes.unicode_minus']=False

plt.figure()

import os
os.environ["CUDA_VISIBLE_DEVICES"] = ""
import  threading
import  gym
import  multiprocessing
import  numpy as np
from    queue import Queue
import  matplotlib.pyplot as plt

import  tensorflow as tf
from    tensorflow import keras
from    tensorflow.keras import layers,optimizers,losses



tf.random.set_seed(1231)
np.random.seed(1231)
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '2'
assert tf.__version__.startswith('2.')


class ActorCritic(keras.Model):
    # Actor-Critic模型
    def __init__(self, state_size, action_size):
        super(ActorCritic, self).__init__()
        self.state_size = state_size # 状态向量长度
        self.action_size = action_size # 动作数量
        # 策略网络Actor
        self.dense1 = layers.Dense(128, activation='relu')
        self.policy_logits = layers.Dense(action_size)

        # V网络Critic
        self.dense2 = layers.Dense(128, activation='relu')
        self.values = layers.Dense(1)

    def call(self, inputs):
        # 获得策略分布Pi(a|s)
        x = self.dense1(inputs)
        logits = self.policy_logits(x)
        # 获得v(s)
        v = self.dense2(inputs)
        values = self.values(v)
        return logits, values


def record(episode,
           episode_reward,
           worker_idx,
           global_ep_reward,
           result_queue,
           total_loss,
           num_steps):
    # 统计工具函数
    if global_ep_reward == 0:
        global_ep_reward = episode_reward
    else:
        global_ep_reward = global_ep_reward * 0.99 + episode_reward * 0.01
    print(
        f"{episode} | "
        f"Average Reward: {int(global_ep_reward)} | "
        f"Episode Reward: {int(episode_reward)} | "
        f"Loss: {int(total_loss / float(num_steps) * 1000) / 1000} | "
        f"Steps: {num_steps} | "
        f"Worker: {worker_idx}"
    )
    result_queue.put(global_ep_reward) # 保存回报，传给主线程
    return global_ep_reward

class Memory:
    def __init__(self):
        self.states = []
        self.actions = []
        self.rewards = []

    def store(self, state, action, reward):
        self.states.append(state)
        self.actions.append(action)
        self.rewards.append(reward)

    def clear(self):
        self.states = []
        self.actions = []
        self.rewards = []

class Agent:
    # 智能体，包含了中央参数网络server
    def __init__(self):
        # server优化器，client不需要，直接从server拉取参数
        self.opt = optimizers.Adam(1e-3)
        # 中央模型，类似于参数服务器
        self.server = ActorCritic(4, 2) # 状态向量，动作数量
        self.server(tf.random.normal((2, 4)))
    def train(self):
        res_queue = Queue() # 共享队列
        # 创建各个交互环境
        workers = [Worker(self.server, self.opt, res_queue, i)
                   for i in range(multiprocessing.cpu_count())]
        for i, worker in enumerate(workers):
            print("Starting worker {}".format(i))
            worker.start()
        # 统计并绘制总回报曲线
        returns = []
        while True:
            reward = res_queue.get()
            if reward is not None:
                returns.append(reward)
            else: # 结束标志
                break
        [w.join() for w in workers] # 等待线程退出 

        print(returns)
        plt.figure()
        plt.plot(np.arange(len(returns)), returns)
        # plt.plot(np.arange(len(moving_average_rewards)), np.array(moving_average_rewards), 's')
        plt.xlabel('回合数')
        plt.ylabel('总回报')
        plt.savefig('a3c-tf-cartpole.svg')


class Worker(threading.Thread): 
    def __init__(self,  server, opt, result_queue, idx):
        super(Worker, self).__init__()
        self.result_queue = result_queue # 共享队列
        self.server = server # 中央模型
        self.opt = opt # 中央优化器
        self.client = ActorCritic(4, 2) # 线程私有网络
        self.worker_idx = idx # 线程id
        self.env = gym.make('CartPole-v1').unwrapped
        self.ep_loss = 0.0

    def run(self): 
        mem = Memory() # 每个worker自己维护一个memory
        for epi_counter in range(500): # 未达到最大回合数
            current_state = self.env.reset() # 复位client游戏状态
            mem.clear()
            ep_reward = 0.
            ep_steps = 0  
            done = False
            while not done:
                # 获得Pi(a|s),未经softmax
                logits, _ = self.client(tf.constant(current_state[None, :],
                                         dtype=tf.float32))
                probs = tf.nn.softmax(logits)
                # 随机采样动作
                action = np.random.choice(2, p=probs.numpy()[0])
                new_state, reward, done, _ = self.env.step(action) # 交互 
                ep_reward += reward # 累加奖励
                mem.store(current_state, action, reward) # 记录
                ep_steps += 1 # 计算回合步数
                current_state = new_state # 刷新状态 

                if ep_steps >= 500 or done: # 最长步数500
                    # 计算当前client上的误差
                    with tf.GradientTape() as tape:
                        total_loss = self.compute_loss(done, new_state, mem) 
                    # 计算误差
                    grads = tape.gradient(total_loss, self.client.trainable_weights)
                    # 梯度提交到server，在server上更新梯度
                    self.opt.apply_gradients(zip(grads,
                                                 self.server.trainable_weights))
                    # 从server拉取最新的梯度
                    self.client.set_weights(self.server.get_weights())
                    mem.clear() # 清空Memory 
                    # 统计此回合回报
                    self.result_queue.put(ep_reward)
                    print(self.worker_idx, ep_reward)
                    break
        self.result_queue.put(None) # 结束线程

    def compute_loss(self,
                     done,
                     new_state,
                     memory,
                     gamma=0.99):
        if done:
            reward_sum = 0. # 终止状态的v(终止)=0
        else:
            reward_sum = self.client(tf.constant(new_state[None, :],
                                     dtype=tf.float32))[-1].numpy()[0]
        # 统计折扣回报
        discounted_rewards = []
        for reward in memory.rewards[::-1]:  # reverse buffer r
            reward_sum = reward + gamma * reward_sum
            discounted_rewards.append(reward_sum)
        discounted_rewards.reverse()
        # 获取状态的Pi(a|s)和v(s)
        logits, values = self.client(tf.constant(np.vstack(memory.states),
                                 dtype=tf.float32))
        # 计算advantage = R() - v(s)
        advantage = tf.constant(np.array(discounted_rewards)[:, None],
                                         dtype=tf.float32) - values
        # Critic网络损失
        value_loss = advantage ** 2
        # 策略损失
        policy = tf.nn.softmax(logits)
        policy_loss = tf.nn.sparse_softmax_cross_entropy_with_logits(
                        labels=memory.actions, logits=logits)
        # 计算策略网络损失时，并不会计算V网络
        policy_loss = policy_loss * tf.stop_gradient(advantage)
        # Entropy Bonus  labels标签值（真实值）logits模型的输出
        entropy = tf.nn.softmax_cross_entropy_with_logits(labels=policy,
                                                          logits=logits)
        policy_loss = policy_loss - 0.01 * entropy
        # 聚合各个误差
        total_loss = tf.reduce_mean((0.5 * value_loss + policy_loss))
        return total_loss


if __name__ == '__main__':
    master = Agent()
    master.train()

```
