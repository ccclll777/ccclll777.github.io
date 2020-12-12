---
title: Pytorch强化学习算法实现
date: 2020-12-12 10:54:37
categories: 
- 强化学习
tags:
- 深度学习框架
- python
- pytorch
---
使用Pytorch框架实现了强化学习算法Policy Gradient/DQN/DDGP
<!-- more -->
#  Policy Gradient算法实现
Policy Gradient算法的思想在[另一篇博客](https://ccclll777.github.io/2020/12/07/Reinforcement-Learning-Basic-Theory/#more)中有介绍了，下面是算法的具体实现。

##  Policy网络
两个线性层，中间使用Relu激活函数连接，最后连接softmax输出每个动作的概率。
```python
class PolicyNet(nn.Module):
    def __init__(self,n_states_num,n_actions_num,hidden_size):
        super(PolicyNet, self).__init__()
        self.data = [] #存储轨迹
        #输入为长度为4的向量 输出为向左  向右两个动作
        self.net = nn.Sequential(
            nn.Linear(in_features=n_states_num, out_features=hidden_size, bias=False),
            nn.ReLU(),
            nn.Linear(in_features=hidden_size, out_features=n_actions_num, bias=False),
            nn.Softmax(dim=1)
        )
    def forward(self, inputs):
        # 状态输入s的shape为向量：[4]
        x = self.net(inputs)
        return x
```

##  将状态输入神经网络，选择动作

 - 这里给出了两种实现方式，具体思想就是输入环境的状态，传入policy网络，给出每一个动作的概率，我们需要选择出现概率最大的那个动作，以及他出现的概率值。

```python
    #将状态传入神经网络 根据概率选择动作
    def  choose_action(self,state):
        #将state转化成tensor 并且维度转化为[4]->[1,4]  unsqueeze(0)在第0个维度上田间
        s = torch.Tensor(state).unsqueeze(0)
        prob = self.pi(s)  # 动作分布:[1,2]
        # 从类别分布中采样1个动作, shape: [1] torch.log(prob), 1
        m = torch.distributions.Categorical(prob)  # 生成分布
        action = m.sample()
        return action.item() , m.log_prob(action)

    def choose_action2(self, state):
        # 将state转化成tensor 并且维度转化为[4]->[1,4]  unsqueeze(0)在第0个维度上田间
        s = torch.Tensor(state).unsqueeze(0)
        prob = self.pi(s)  # 动作分布:[1,2]
        # 从类别分布中采样1个动作, shape: [1] torch.log(prob), 1
        action =np.random.choice(range(prob.shape[1]),size=1,p = prob.view(-1).detach().numpy())[0]
        action = int(action)
        #print(torch.log(prob[0][action]).unsqueeze(0))
        return action,torch.log(prob[0][action]).unsqueeze(0)
```

 - （1）使用`torch.distributions.Categorical`生成分布，然后进行选择
 - （2）使用`np.random.choice`进行采样
##  模型的训练

```python
    def train_net(self):
        # 计算梯度并更新策略网络参数。tape为梯度记录器
        R = 0  # 终结状态的初始回报为0
        policy_loss = []
        for r, log_prob in self.data[::-1]:  # 逆序取
            R = r + gamma * R  # 计算每个时间戳上的回报
            # 每个时间戳都计算一次梯度
            loss = -log_prob * R
            policy_loss.append(loss)
        self.optimizer.zero_grad()
        policy_loss = torch.cat(policy_loss).sum()  # 求和
        #反向传播
        policy_loss.backward()
        self.optimizer.step()
        self.cost_his.append(policy_loss.item())
        self.data = []  # 清空轨迹
```
##  完整代码

```python

import 	gym,os
import  numpy as np
import  matplotlib
# Default parameters for plots
matplotlib.rcParams['font.size'] = 18
matplotlib.rcParams['figure.titlesize'] = 18
matplotlib.rcParams['figure.figsize'] = [9, 7]
matplotlib.rcParams['font.family'] = ['KaiTi']
matplotlib.rcParams['axes.unicode_minus']=False

import torch
from torch import nn
env = gym.make('CartPole-v1')
env.seed(2333)
torch.manual_seed(2333)    # 策略梯度算法方差很大，设置seed以保证复现性
print('observation space:',env.observation_space)
print('action space:',env.action_space)

learning_rate = 0.0002
gamma         = 0.98

class PolicyNet(nn.Module):
    def __init__(self,n_states_num,n_actions_num,hidden_size):
        super(PolicyNet, self).__init__()
        self.data = [] #存储轨迹
        #输入为长度为4的向量 输出为向左  向右两个动作
        self.net = nn.Sequential(
            nn.Linear(in_features=n_states_num, out_features=hidden_size, bias=False),
            nn.ReLU(),
            nn.Linear(in_features=hidden_size, out_features=n_actions_num, bias=False),
            nn.Softmax(dim=1)
        )

    def forward(self, inputs):
        # 状态输入s的shape为向量：[4]
        x = self.net(inputs)
        return x

class PolicyGradient():
    def __init__(self,n_states_num,n_actions_num,learning_rate=0.01,reward_decay=0.95 ):
        #状态数   state是一个4维向量，分别是位置，速度，杆子的角度，加速度
        self.n_states_num = n_states_num
        #action是二维、离散，即向左/右推杆子
        self.n_actions_num = n_actions_num
        #学习率
        self.lr = learning_rate
        #gamma
        self.gamma = reward_decay
        #网络
        self.pi = PolicyNet(n_states_num, n_actions_num, 128)
        #优化器
        self.optimizer = torch.optim.Adam(self.pi.parameters(), lr=learning_rate)
        # 存储轨迹  存储方式为  （每一次的reward，动作的概率）
        self.data = []
        self.cost_his = []
    #存储轨迹数据
    def put_data(self, item):
        # 记录r,log_P(a|s)z
        self.data.append(item)
    def train_net(self):
        # 计算梯度并更新策略网络参数。tape为梯度记录器
        R = 0  # 终结状态的初始回报为0
        policy_loss = []
        for r, log_prob in self.data[::-1]:  # 逆序取
            R = r + gamma * R  # 计算每个时间戳上的回报
            # 每个时间戳都计算一次梯度
            loss = -log_prob * R
            policy_loss.append(loss)
        self.optimizer.zero_grad()
        policy_loss = torch.cat(policy_loss).sum()  # 求和
        #反向传播
        policy_loss.backward()
        self.optimizer.step()
        self.cost_his.append(policy_loss.item())
        self.data = []  # 清空轨迹
    #将状态传入神经网络 根据概率选择动作
    def  choose_action(self,state):
        #将state转化成tensor 并且维度转化为[4]->[1,4]  unsqueeze(0)在第0个维度上田间
        s = torch.Tensor(state).unsqueeze(0)
        prob = self.pi(s)  # 动作分布:[1,2]
        # 从类别分布中采样1个动作, shape: [1] torch.log(prob), 1
        m = torch.distributions.Categorical(prob)  # 生成分布
        action = m.sample()
        return action.item() , m.log_prob(action)

    def choose_action2(self, state):
        # 将state转化成tensor 并且维度转化为[4]->[1,4]  unsqueeze(0)在第0个维度上田间
        s = torch.Tensor(state).unsqueeze(0)
        prob = self.pi(s)  # 动作分布:[1,2]
        # 从类别分布中采样1个动作, shape: [1] torch.log(prob), 1
        action =np.random.choice(range(prob.shape[1]),size=1,p = prob.view(-1).detach().numpy())[0]
        action = int(action)
        #print(torch.log(prob[0][action]).unsqueeze(0))
        return action,torch.log(prob[0][action]).unsqueeze(0)

    def plot_cost(self):
        import matplotlib.pyplot as plt
        plt.plot(np.arange(len(self.cost_his)), self.cost_his)
        plt.ylabel('Cost')
        plt.xlabel('training steps')
        plt.show()

def main():
    policyGradient = PolicyGradient(4,2)
    running_reward = 10  # 计分
    print_interval = 20  # 打印间隔
    for n_epi in range(1000):
        state = env.reset()  # 回到游戏初始状态，返回s0
        ep_reward = 0
        for t in range(1001):  # CartPole-v1 forced to terminates at 1000 step.
            #根据状态 传入神经网络 选择动作
            action ,log_prob  = policyGradient.choose_action2(state)
            #与环境交互
            s_prime, reward, done, info = env.step(action)
            # s_prime, reward, done, info = env.step(action)
            if n_epi > 1000:
                env.render()
            # 记录动作a和动作产生的奖励r
            # prob shape:[1,2]
            policyGradient.put_data((reward, log_prob))
            state = s_prime  # 刷新状态
            ep_reward += reward
            if done:  # 当前episode终止
                break
            # episode终止后，训练一次网络
        running_reward = 0.05 * ep_reward + (1 - 0.05) * running_reward
        #交互完成后 进行学习
        policyGradient.train_net()
        if n_epi % print_interval == 0:
            print('Episode {}\tLast reward: {:.2f}\tAverage reward: {:.2f}'.format(
                n_epi, ep_reward, running_reward))
        if running_reward > env.spec.reward_threshold:  # 大于游戏的最大阈值475时，退出游戏
            print("Solved! Running reward is now {} and "
                  "the last episode runs to {} time steps!".format(running_reward, t))
            break
    policyGradient.plot_cost()

if __name__ == '__main__':
    main()

```

#  DQN算法实现
DQN算法的思想在[另一篇博客](https://ccclll777.github.io/2020/12/07/Reinforcement-Learning-Basic-Theory/#more)中有介绍了，下面是算法的具体实现。
##  经验回放池
这里使用python的双向队列实现了经验回放池，实现了状态的存储以及随机采样。

 - 经验回放 Experience replay：由于在强化学习中，我们得到 的观测数据是有序的，用这样的数据去更新神经网络的参数会有问题（对比监督学习，数据之间都是独立的）。因此 DQN 中使 用经验回放，即用一个 Memory 来存储经历过的数据，每次更新参数的时候从 Memory 中抽取一部分的数据来用于更新，以此来打破数据间的关联。

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
        s_lst, a_lst, r_lst, s_prime_lst = [], [], [], []
        # 按类别进行整理
        for transition in mini_batch:
            s, a, r, s_prime = transition
            s_lst.append(s)
            a_lst.append([a])
            r_lst.append([r])
            s_prime_lst.append(s_prime)
        # 转换成Tensor
        return torch.Tensor(s_lst), \
               torch.Tensor(a_lst), \
                      torch.Tensor(r_lst), \
                      torch.Tensor(s_prime_lst)
    def size(self):
        return len(self.buffer)
```
##  Q网络
DQN使用神经网络取代了Q Table去预测Q（s，a）

```python
class Qnet(nn.Module):
    def __init__(self,input_size,output_size,hidden_size):
        # 创建Q网络，输入为状态向量，输出为动作的Q值
        super(Qnet, self).__init__()
        self.net = nn.Sequential(
            nn.Linear(in_features=input_size, out_features=hidden_size),
            nn.ReLU(),
            nn.Linear(in_features=hidden_size, out_features=hidden_size),
            nn.ReLU(),
            nn.Linear(in_features=hidden_size, out_features=output_size),
        )
    def forward(self, inputs, training=None):
        x = self.net(inputs)
        return x
```
##  模型训练
模型使用的两个Q网络，  在原来的 Q 网络的基础上又引入了一个 target Q 网络，即用来计算 target 的网络。它和 Q 网络结构一样， 初始的权重也一样，只是 Q 网络每次迭代都会更新，而 target Q 网络是每隔一段时间才会更新。

```python
#训练模型
    def train(self):
        if self.learn_step_counter % self.replace_target_iter == 0:
            self.q_target_net.load_state_dict(self.q_net.state_dict())

        # 通过Q网络和影子网络来构造贝尔曼方程的误差，
        # 并只更新Q网络，影子网络的更新会滞后Q网络
        for i in range(10): # 训练10次
            s, a, r, s_prime = self.memory.sample(self.batch_size)
            # q_prime  用旧网络、动作后的环境预测，q_a 用新网络、动作前的环境；同时预测记忆中的情形
            q_next, q_eval = self.q_target_net(s),self.q_net(s_prime)
            # 每次学习都用下一个状态的动作结合反馈作为当前动作值（这样，将未来状态的动作作为目标，有一定前瞻性）
            q_target = q_eval
            #action的index值 它所在的位置

            #实际Q网络的值
            act_index = np.array(a.tolist()).astype(np.int64)
            act_index = torch.from_numpy(act_index)
            q_a = q_eval.gather(1, act_index)  # 动作的概率值, [b,1]

            #target Q网络的值
            max_q_prime, _ = torch.max(q_next, dim=1)
            max_q_prime = max_q_prime.unsqueeze(1)

            q_target = r + self.gamma * max_q_prime
            #q_out[batch_index, eval_act_index] = reward + self.gamma * np.max(q_prime, axis=1)


            loss = self.loss(q_a,q_target)
            self.optimizer.zero_grad()
            loss.backward()
            self.optimizer.step()
            cost = loss.item()

            self.cost_his.append(cost)

            self.epsilon = self.epsilon + self.epsilon_increment if self.epsilon < self.epsilon_max else self.epsilon_max
            self.learn_step_counter += 1
```
##  完整代码

```python
import collections
import random
import gym,os
import  numpy as np
import torch
from torch import nn
import torch.nn.functional as F
env = gym.make('CartPole-v1')
env.seed(2333)
torch.manual_seed(2333)    # 策略梯度算法方差很大，设置seed以保证复现性
print('observation space:',env.observation_space)
print('action space:',env.action_space)

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
        s_lst, a_lst, r_lst, s_prime_lst = [], [], [], []
        # 按类别进行整理
        for transition in mini_batch:
            s, a, r, s_prime = transition
            s_lst.append(s)
            a_lst.append([a])
            r_lst.append([r])
            s_prime_lst.append(s_prime)
        # 转换成Tensor
        return torch.Tensor(s_lst), \
               torch.Tensor(a_lst), \
                      torch.Tensor(r_lst), \
                      torch.Tensor(s_prime_lst)


    def size(self):
        return len(self.buffer)


class Qnet(nn.Module):
    def __init__(self,input_size,output_size,hidden_size):
        # 创建Q网络，输入为状态向量，输出为动作的Q值
        super(Qnet, self).__init__()
        self.net = nn.Sequential(
            nn.Linear(in_features=input_size, out_features=hidden_size),
            nn.ReLU(),
            nn.Linear(in_features=hidden_size, out_features=hidden_size),
            nn.ReLU(),
            nn.Linear(in_features=hidden_size, out_features=output_size),
        )

    def forward(self, inputs, training=None):
        x = self.net(inputs)
        return x

class DQN():
    def __init__(self,input_size,output_size,hidden_size,learning_rate,reward_decay,epsilon,e_greedy_increment,e_greedy):
        self.n_actions = output_size
        self.n_features = input_size
        self.learning_rate = learning_rate
        self.gamma = reward_decay
        self.epsilon = epsilon #e - 贪心方式 参数
        self.q_net = Qnet(input_size,output_size,hidden_size)
        self.q_target_net = Qnet(input_size,output_size,hidden_size)  # 创建影子网络
        self.q_target_net.load_state_dict(self.q_net.state_dict()) # 影子网络权值来自Q
        self.optimizer = torch.optim.Adam(self.q_net.parameters(), lr=learning_rate)#优化器
        self.buffer_limit = 50000
        self.batch_size = 32
        self.memory = ReplayBuffer()  #创建回放池
        # Huber Loss常用于回归问题，其最大的特点是对离群点（outliers）、噪声不敏感，具有较强的鲁棒性
        self.loss  = torch.nn.SmoothL1Loss()#损失函数
        self.cost_his = []
        self.epsilon_increment =e_greedy_increment
        self.learn_step_counter = 0
        self.epsilon_max = e_greedy

        self.replace_target_iter = 300
    # 送入状态向量，获取策略: [4]
    def choose_action(self,state):
        # 将state转化成tensor 并且维度转化为[4]->[1,4]  unsqueeze(0)在第0个维度上田间
        state = torch.Tensor(state).unsqueeze(0)

        # 策略改进：e-贪心方式
        if np.random.uniform() < self.epsilon:
            actions_value = self.q_net(state)
            action = np.argmax(actions_value.detach().numpy())
        else:
            action = np.random.randint(0, self.n_actions)
        return action
    #训练模型
    def train(self):
        if self.learn_step_counter % self.replace_target_iter == 0:
            self.q_target_net.load_state_dict(self.q_net.state_dict())
        # 通过Q网络和影子网络来构造贝尔曼方程的误差，
        # 并只更新Q网络，影子网络的更新会滞后Q网络
        for i in range(10): # 训练10次
            s, a, r, s_prime = self.memory.sample(self.batch_size)
            # q_prime  用旧网络、动作后的环境预测，q_a 用新网络、动作前的环境；同时预测记忆中的情形
            q_next, q_eval = self.q_target_net(s),self.q_net(s_prime)
            # 每次学习都用下一个状态的动作结合反馈作为当前动作值（这样，将未来状态的动作作为目标，有一定前瞻性）
            q_target = q_eval
            #action的index值 它所在的位置
            #实际Q网络的值
            act_index = np.array(a.tolist()).astype(np.int64)
            act_index = torch.from_numpy(act_index)
            q_a = q_eval.gather(1, act_index)  # 动作的概率值, [b,1]
            #target Q网络的值
            max_q_prime, _ = torch.max(q_next, dim=1)
            max_q_prime = max_q_prime.unsqueeze(1)
            q_target = r + self.gamma * max_q_prime
            loss = self.loss(q_a,q_target)
            self.optimizer.zero_grad()
            loss.backward()
            self.optimizer.step()
            cost = loss.item()

            self.cost_his.append(cost)

            self.epsilon = self.epsilon + self.epsilon_increment if self.epsilon < self.epsilon_max else self.epsilon_max
            self.learn_step_counter += 1
    def plot_cost(self):
        import matplotlib.pyplot as plt
        plt.plot(np.arange(len(self.cost_his)), self.cost_his)
        plt.ylabel('Cost')
        plt.xlabel('training steps')
        plt.show()
def main():
    env = gym.make('CartPole-v1')  # 创建环境
    dqn = DQN(input_size = 4,
              output_size = 2,
              hidden_size = 10,
              learning_rate = 0.01,
              reward_decay=0.9,epsilon=0.9,
              e_greedy_increment=0.001,
              e_greedy=0.9,
              )

    print_interval = 20
    reword = 0.0
    for n_epi in range(2000):  # 训练次数
        # epsilon概率也会8%到1%衰减，越到后面越使用Q值最大的动作
        dqn.epsilon = max(0.01, 0.08 - 0.01 * (n_epi / 200))
        state = env.reset()  # 复位环境
        while True: # 一个回合最大时间戳
            # 根据当前Q网络提取策略，并改进策略
            action = dqn.choose_action(state)
            # 使用改进的策略与环境交互
            s_prime, r, done, info = env.step(action)
            #https://blog.csdn.net/u012465304/article/details/81172759
            #由于CartPole这个游戏的reward是只要杆子是立起来的，他reward就是1，失败就是0，
            # 显然这个reward对于连续性变量是不可以接受的，所以我们通过observation修改这个值。
            # 点击pycharm右上角的搜索符号搜索CartPole进入他环境的源代码中，再进入step函数，
            # 看到里面返回值state的定义
            #通过这四个值定义新的reward是
            x, x_dot, theta, theta_dot = s_prime
            r1 = (env.x_threshold - abs(x)) / env.x_threshold - 0.8
            r2 = (env.theta_threshold_radians - abs(theta)) / env.theta_threshold_radians - 0.5
            reward = r1 + r2

            # 保存四元组
            dqn.memory.put((state, action,reward,s_prime))
            state = s_prime
            reword +=reward
            if done:  # 回合结束
                break

            if dqn.memory.size() > 1000:  # 缓冲池只有大于2000就可以训练
                dqn.train()
        if n_epi % print_interval == 0 and n_epi != 0:
            print("# of episode :{}, avg score : {:.1f}, buffer size : {}, " \
                  "epsilon : {:.1f}%" \
                  .format(n_epi, reword / print_interval, dqn.memory.size(), dqn.epsilon * 100))
            reword = 0.0
    env.close()
    dqn.plot_cost()
if __name__ == "__main__":
    main()



```
#  DDPG算法实现
DDPG算法的思想在[另一篇博客](https://ccclll777.github.io/2020/12/07/Reinforcement-Learning-Basic-Theory/#more)中有介绍了，下面是算法的具体实现。

DDPG 可以解决连续动作空间问题，并且是actor-critic方法，即既有值函数网络(critic)，又有策略网络(actor)。
##  经验回放池
与DQN中的经验回放池的实现相同
```python
class ReplayBuffer():
    # 经验回放池
    def __init__(self):
        # 双向队列
        buffer_limit = 50000
        self.buffer = collections.deque(maxlen=buffer_limit)
        #通过 put(transition)方法 将最新的(𝑠, 𝑎, 𝑟, 𝑠′)数据存入 Deque 对象
    def put(self, transition):
        self.buffer.append(transition)
    #通过 sample(n)方法从 Deque 对象中随机采样出 n 个(𝑠, 𝑎, 𝑟, 𝑠′)数据
    def sample(self, n):
        # 从回放池采样n个5元组
        mini_batch = random.sample(self.buffer, n)
        s_lst, a_lst, r_lst, s_prime_lst = [], [], [], []
        # 按类别进行整理
        for transition in mini_batch:
            s, a, r, s_prime = transition
            s_lst.append(s)
            a_lst.append([a])
            r_lst.append([r])
            s_prime_lst.append(s_prime)
        # 转换成Tensor
        return torch.Tensor(s_lst), \
               torch.Tensor(a_lst), \
                      torch.Tensor(r_lst), \
                      torch.Tensor(s_prime_lst)
    def size(self):
        return len(self.buffer)
```
##  策略网络（Actor网络）
输入为state  输出为概率分布pi(a|s)（每个动作出现的概率）

```python
# 策略网络，也叫Actor网络，输入为state  输出为概率分布pi(a|s)
class Actor(nn.Module):
    def __init__(self,input_size,hidden_size,output_size):
        super(Actor, self).__init__()
        # self.linear  = nn.Linear(hidden_size, output_size)
        self.actor_net = nn.Sequential(
            nn.Linear(in_features=input_size,out_features=hidden_size),
            nn.ReLU(),
            nn.Linear(in_features=hidden_size,out_features=hidden_size),
            nn.ReLU(),
            nn.Linear(in_features=hidden_size,out_features=output_size)
        )
    def forward(self,state):
        x = self.actor_net(state)
        x = torch.tanh(x)
        return x
```
##  值函数网络  输入是state，action输出是Q(s,a)

```python
#值函数网络  输入是state，action输出是Q(s,a)
class Critic(nn.Module):
    def __init__(self, input_size, hidden_size, output_size):
        super(Critic, self).__init__()
        self.critic_net = nn.Sequential(
            nn.Linear(in_features=input_size, out_features=hidden_size),
            nn.ReLU(),
            nn.Linear(in_features=hidden_size, out_features=hidden_size),
            nn.ReLU(),
            nn.Linear(in_features=hidden_size, out_features=output_size)
        )
    def forward(self, state,action):
        inputs = torch.cat([state,action],1)
        x = self.critic_net(inputs)
        return x
```
##  整体实现
```python
import gym
import random
import numpy as np
import torch
import torch.nn as nn
import torch.optim as optim
import  collections

env = gym.make('Pendulum-v0')
env.seed(2333)
torch.manual_seed(2333)    # 策略梯度算法方差很大，设置seed以保证复现性
env.reset()
env.render()
print('observation space:',env.observation_space)
print('action space:',env.action_space)
class ReplayBuffer():
    # 经验回放池
    def __init__(self):
        # 双向队列
        buffer_limit = 50000
        self.buffer = collections.deque(maxlen=buffer_limit)
        #通过 put(transition)方法 将最新的(𝑠, 𝑎, 𝑟, 𝑠′)数据存入 Deque 对象
    def put(self, transition):
        self.buffer.append(transition)
    #通过 sample(n)方法从 Deque 对象中随机采样出 n 个(𝑠, 𝑎, 𝑟, 𝑠′)数据
    def sample(self, n):
        # 从回放池采样n个5元组
        mini_batch = random.sample(self.buffer, n)
        s_lst, a_lst, r_lst, s_prime_lst = [], [], [], []
        # 按类别进行整理
        for transition in mini_batch:
            s, a, r, s_prime = transition
            s_lst.append(s)
            a_lst.append([a])
            r_lst.append([r])
            s_prime_lst.append(s_prime)
        # 转换成Tensor
        return torch.Tensor(s_lst), \
               torch.Tensor(a_lst), \
                      torch.Tensor(r_lst), \
                      torch.Tensor(s_prime_lst)


    def size(self):
        return len(self.buffer)


# 策略网络，也叫Actor网络，输入为state  输出为概率分布pi(a|s)
class Actor(nn.Module):
    def __init__(self,input_size,hidden_size,output_size):
        super(Actor, self).__init__()
        # self.linear  = nn.Linear(hidden_size, output_size)
        self.actor_net = nn.Sequential(
            nn.Linear(in_features=input_size,out_features=hidden_size),
            nn.ReLU(),
            nn.Linear(in_features=hidden_size,out_features=hidden_size),
            nn.ReLU(),
            nn.Linear(in_features=hidden_size,out_features=output_size)
        )
    def forward(self,state):
        x = self.actor_net(state)
        x = torch.tanh(x)
        return x

#值函数网络  输入是state，action输出是Q(s,a)
class Critic(nn.Module):
    def __init__(self, input_size, hidden_size, output_size):
        super(Critic, self).__init__()
        self.critic_net = nn.Sequential(
            nn.Linear(in_features=input_size, out_features=hidden_size),
            nn.ReLU(),
            nn.Linear(in_features=hidden_size, out_features=hidden_size),
            nn.ReLU(),
            nn.Linear(in_features=hidden_size, out_features=output_size)
        )

    def forward(self, state,action):
        inputs = torch.cat([state,action],1)
        x = self.critic_net(inputs)
        return x


class DDPG():
    def __init__(self,state_size,action_size,hidden_size = 256,actor_lr = 0.001,ctitic_lr = 0.001,batch_size = 32):

        self.state_size = state_size
        self.action_size = action_size
        self.hidden_size = hidden_size
        self.actor_lr = actor_lr #actor网络学习率
        self.critic_lr = ctitic_lr#critic网络学习率
        # 策略网络，也叫Actor网络，输入为state  输出为概率分布pi(a|s)
        self.actor = Actor(self.state_size, self.hidden_size, self.action_size)
        #target actor网络 延迟更新
        self.actor_target = Actor(self.state_size, self.hidden_size, self.action_size)
        # 值函数网络  输入是state，action输出是Q(s,a)
        self.critic = Critic(self.state_size + self.action_size, self.hidden_size, self.action_size)
        self.critic_target = Critic(self.state_size + self.action_size, self.hidden_size, self.action_size)

        self.actor_optim = optim.Adam(self.actor.parameters(), lr=self.actor_lr)
        self.critic_optim = optim.Adam(self.critic.parameters(), lr=self.critic_lr)
        self.buffer = []
        # 影子网络权值来自原网络，只不过延迟更新
        self.actor_target.load_state_dict(self.actor.state_dict())
        self.critic_target.load_state_dict(self.critic.state_dict())
        self.gamma = 0.99
        self.batch_size = batch_size
        self.memory = ReplayBuffer()  # 创建回放池

        self.memory2 = []
        self.learn_step_counter = 0 #学习轮数 与影子网络的更新有关
        self.replace_target_iter = 200 #影子网络迭代多少轮更新一次
        self.cost_his_actor = []# 存储cost 准备画图
        self.cost_his_critic = []


    def choose_action(self,state):
        # 将state转化成tensor 并且维度转化为[3]->[1,3]  unsqueeze(0)在第0个维度上田间
        state = torch.Tensor(state).unsqueeze(0)
        action = self.actor(state).squeeze(0).detach().numpy()
        return action
    #critic网络的学习
    def critic_learn(self,s0,a0,r1,s1):
        #从actor_target通过状态获取对应的动作  detach()将tensor从计算图上剥离
        a1 = self.actor_target(s0).detach()
        #删减一个维度  [b,1,1]变成[b,1]
        a0 = a0.squeeze(2)
        y_pred = self.critic(s0,a0)
        y_target = r1 +self.gamma *self.critic_target(s1,a1).detach()
        loss_fn = nn.MSELoss()
        loss = loss_fn(y_pred, y_target)
        self.critic_optim.zero_grad()
        loss.backward()
        self.critic_optim.step()
        self.cost_his_critic.append(loss.item())
    #actor网络的学习
    def actor_learn(self,s0,a0,r1,s1):
        loss = -torch.mean(self.critic(s0, self.actor(s0)))
        self.actor_optim.zero_grad()
        loss.backward()
        self.actor_optim.step()
        self.cost_his_actor.append(loss.item())
    #模型的训练
    def train(self):
        if self.learn_step_counter % self.replace_target_iter == 0:
            self.actor_target.load_state_dict(self.actor.state_dict())
            self.critic_target.load_state_dict(self.critic.state_dict())
        #随机采样出 batch_size 个(𝑠, 𝑎, 𝑟, 𝑠′)数据
        s0, a0, r, s_prime = self.memory.sample(self.batch_size)
        self.critic_learn(s0, a0, r, s_prime)
        self.actor_learn(s0, a0, r, s_prime)

        self.soft_update(self.critic_target, self.critic, 0.02)
        self.soft_update(self.actor_target, self.actor, 0.02)
    #target网络的更新
    def soft_update(self,net_target, net, tau):
        for target_param, param in zip(net_target.parameters(), net.parameters()):
            target_param.data.copy_(target_param.data * (1.0 - tau) + param.data * tau)

    def plot_cost(self):
        import matplotlib.pyplot as plt
        plt.plot(np.arange(len(self.cost_his_critic)), self.cost_his_critic)
        plt.ylabel('Cost')
        plt.xlabel('training steps')
        plt.show()



def main():
    print(env.observation_space.shape[0])
    print(env.action_space.shape[0])
    ddgp = DDPG(state_size=env.observation_space.shape[0],
                action_size=env.action_space.shape[0],
                hidden_size=256,
                actor_lr=0.001,
                ctitic_lr=  0.001,
                batch_size=32)

    print_interval = 4

    for episode in range(100):
        state = env.reset()
        episode_reward = 0

        for step in range(500):
            env.render()
            action0 = ddgp.choose_action(state)
            s_prime, r, done, info = env.step(action0)

            # 保存四元组
            ddgp.memory.put((state, action0, r, s_prime))
            episode_reward += r
            state = s_prime

            if done:  # 回合结束
                break

            if ddgp.memory.size() > 32:  # 缓冲池只有大于500就可以训练
                ddgp.train()

        if episode % print_interval == 0 and episode != 0:
            print("# of episode :{}, avg score : {:.1f}, buffer size : {}, "
                  .format(episode, episode_reward / print_interval, ddgp.memory.size()))
    env.close()
    ddgp.plot_cost()

if __name__ == "__main__":
    main()
```
