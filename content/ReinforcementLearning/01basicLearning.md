---
title: 🎃 基础强化学习
tags:
  - ReinforcementLearning
  - 强化学习
---

# 强化学习：从 MDP 到 Q‑Learning

#强化学习 #MDP #Q-learning

## 一、强化学习的核心思想

强化学习（Reinforcement Learning, RL）的目标是让智能体（Agent）在与环境（Environment）的交互中，通过不断试错来学习最优策略（Policy），从而最大化长期累积奖励。
- **智能体**：学习和决策的主体。
- **环境**：智能体所处的外部世界，会根据智能体的动作发生变化。
- **策略 $\pi$**：智能体的行为函数，即 “在状态 s 下，选择动作 a 的概率”$\pi(a|s)$。
- **目标**：最大化未来奖励的期望总和。
## 二、马尔可夫决策过程（MDP）：RL 的数学框架

MDP 是对强化学习问题的标准数学抽象，它假设环境的下一个状态只依赖于当前状态和动作，而与更早的历史无关（马尔可夫性）。
### 1. MDP 五元组 $(S,A,P,R,\gamma)$
| 符号       | 名称     | 含义                                              |
| -------- | ------ | ----------------------------------------------- |
| S        | 状态空间   | 所有可能状态的集合。                                      |
| A        | 动作空间   | 所有可选动作的集合。                                      |
| P        | 状态转移函数 | $P(s' \mid s,a)$，在状态 s 执行动作 a 后，转移到状态 $s'$ 的概率。 |
| R        | 奖励函数   | $R(s,a,s')$，在状态 s 执行动作 a 并转移到 $s'$ 后获得的即时奖励。    |
| $\gamma$ | 折扣因子   | $0 \le \gamma \le 1$，用于权衡眼前奖励和未来奖励的重要性。         |
### 2. 价值函数（Value Function）

价值函数衡量一个状态（或状态‑动作对）的 “好坏” 程度，即从该状态开始，遵循某个策略所能获得的期望累积奖励。
- **状态价值函数$V^\pi(s)$**：
$V^\pi(s)=\mathbb{E}_\pi\left[\sum_{k=0}^{\infty}\gamma^kR_{t+k+1}\mid S_t=s\right]$

表示在状态 s 下，遵循策略 $\pi$ 所能获得的期望长期回报。
- 动作价值函数 $Q^\pi(s,a)$
$Q^\pi(s, a) = \mathbb{E}_\pi\left[\sum_{k=0}^{\infty}\gamma^k R_{t+k+1} \mid S_t = s,\, A_t = a\right]$

表示在状态 s 下执行动作 a，之后遵循策略 $\pi$ 所能获得的期望长期回报。
## 三、贝尔曼方程：价值的递推关系

贝尔曼方程是价值函数的核心递推式，它将当前价值与未来价值联系起来，是所有强化学习算法的理论基础。
### 1. 贝尔曼期望方程（评估当前策略）

用于计算在给定策略 $\pi$ 下，每个状态的价值。
- 对于状态价值函数：
$V^\pi(s) = \sum_{a}\pi(a|s)\sum_{s'}P(s'|s,a)\left[R(s,a,s')+\gamma V^\pi(s')\right]$
- 对于动作价值函数：
$Q^\pi(s,a) = \sum_{s'}P(s'|s,a)\left[R(s,a,s')+\gamma\sum_{a'}\pi(a'|s')Q^\pi(s',a')\right]$
### 2. 贝尔曼最优方程（寻找最优策略）

最优价值函数 $V^*(s)$ 和 $Q^*(s,a)$ 是在所有可能策略中能获得的最大价值。
- 最优状态价值函数：
$V^{*}(s) = \max_{a}\sum_{s'}P(s'|s,a)\left[R(s,a,s')+\gamma V^{*}(s')\right]$
- 最优动作价值函数：
$Q^{*}(s,a) = \sum_{s'}P(s'|s,a)\left[R(s,a,s')+\gamma\max_{a'}Q^{*}(s',a')\right]$

> 关键点：$\max$ 算子意味着，在未来的每一步，我们都会选择能带来最大价值的动作，这定义了最优策略。
## 四、Q‑Learning：一种经典的无模型算法

当环境的转移概率 P 未知时，我们无法直接使用动态规划（DP）方法，这时就需要无模型（Model‑free）算法，如 Q‑Learning。
### 1. 核心思想

Q‑Learning 的目标是直接学习最优动作价值函数 $Q^{*}(s,a)$，而无需先学习环境模型。它通过与环境交互，采样得到 $(s,a,r,s')$ 四元组，然后用这些样本来更新 Q 表。
### 2. Q‑Learning 更新公式

$Q(s, a) \leftarrow Q(s, a) + \alpha\left[R+\gamma\max_{a'}Q(s',a')-Q(s,a)\right]$
- 学习率 $\alpha$：控制每次更新的步长，$\alpha$ 越大，新信息覆盖旧信息的速度越快。
- 目标值：由即时奖励 $R$ 和对未来最优价值的估计 $\gamma\max_{a'}Q(s',a')$ 组成。
- 更新过程：用时序差分误差（TD Error）来修正旧估计。
### 3. 算法流程

1. 初始化：创建一个 Q 表，将所有 $Q(s,a)$ 的值初始化为 0 或小的随机数。
2. 循环 episodes：
3. 初始化状态 $s$。
4. 循环 steps：
    1. 使用 $\epsilon$‑贪心策略选择动作 $a$。
    2. 执行动作 $a$，得到奖励 $r$ 和下一个状态 $s'$。
    3. 根据公式更新 Q 表。
    4. 更新当前状态 $s \leftarrow s'$。
    5. 如果 $s$ 是终止状态，则结束该 episode。
## 五、Q‑Learning 代码示例（Python）
```python
import gymnasium as gym
import numpy as np

# 1. 创建环境 (注意使用新的库名 gymnasium)
env = gym.make("CliffWalking‑v0")

# 超参数
alpha = 0.1   # 学习率
gamma = 0.99  # 折扣因子
epsilon = 0.1 # 探索率
episodes = 500

# 初始化 Q 表: [状态数, 动作数]
# CliffWalking 有 48 个状态 (4x12 栅格), 4 个动作
Q = np.zeros([env.observation_space.n, env.action_space.n])

for episode in range(episodes):
    # 最新 API: reset 返回 (state, info)
    state, info = env.reset()
    done = False

    while not done:
        # epsilon‑greedy 策略选择动作
        if np.random.uniform(0, 1) < epsilon:
            action = env.action_space.sample()  # 探索
        else:
            action = np.argmax(Q[state])        # 利用

        # 最新 API: step 返回 5 个值
        # terminated: 掉进悬崖或到达终点
        # truncated: 步数达到上限（默认通常是 100 步）
        next_state, reward, terminated, truncated, info = env.step(action)

        # 只要其中一个为 True，本轮结束
        done = terminated or truncated

        # 核心更新公式 (TD Error / 贝尔曼方程)
        # Q(s,a) = Q(s,a) + alpha * [R + gamma * max(Q(s')) - Q(s,a)]
        best_next_action = np.max(Q[next_state])
        td_target = reward + gamma * best_next_action
        Q[state, action] += alpha * (td_target - Q[state, action])

        state = next_state

    if (episode + 1) % 50 == 0:
        print(f"Episode {episode + 1} 训练中...")

print("训练完成！")
````


# 强化学习：从 Policy Gradient 到 REINFORCE

#强化学习 #策略梯度 #Policy-Gradient #REINFORCE
## 一、为什么需要 Policy Gradient?

上一篇介绍了 Value‑based 方法（Q‑Learning / DQN）：先学价值函数 $Q(s,a)$，再通过 $\arg\max_{a}Q$ 推导策略。这条路线简洁有效，但存在三个根本性的瓶颈

| 瓶颈 | 举例说明 |
| ---- | ---- |
| 连续动作空间无解 | 机械臂力矩、自驾方向盘角度等连续值，无法穷举所有 $a$ 取 $\arg\max$ |
| 只能学确定性策略 | $\arg\max Q$ 输出唯一动作；而石头剪刀布、部分可观测等场景中，随机策略才是最优解 |
| $\arg\max$ 不可微 | 离散动作上的 max 操作无法反向传播，限制了端到端优化 |

Policy Gradient 的思路完全不同：跳过价值函数，直接用一个参数化网络 $\pi_\theta(a|s)$ 输出动作的概率分布，然后对策略参数 $\theta$ 做梯度上升来最大化期望回报。
- Value‑based: $s \rightarrow Q(s,a) \rightarrow argmax \rightarrow a$（间接）
- Policy‑based: $s \rightarrow \pi_\theta(a|s) \rightarrow sample \rightarrow a$（直接）
这带来了三个天然优势：
- 连续动作：输出概率分布，如高斯分布 $\mathcal{N}\left(\mu_\theta(s),\sigma_\theta(s)\right)$，直接采样，无需离散化，且参数简洁
- 随机策略：概率分布本身包含随机性，天然适配探索与博弈场景
- 平滑收敛：$\theta$ 的小扰动只导致动作概率的小变化，不像 $\arg\max$ 那样发生策略跳变
## 二、核心数学推导
### 2.1 目标函数
我们要找到最优参数 $\theta^*$，使得策略的期望回报最大：
$$
J(\theta) = \mathbb{E}_{\tau\sim\pi_\theta}[R(\tau)]
$$
- $\tau = (s_0,a_0,r_0,s_1,a_1,r_1,\dots,s_{T-1},a_{T-1},r_{T-1})$：一条完整轨迹
- $R(\tau)=\sum_{t=0}^{T-1}\gamma^t r_t$：轨迹的折扣总奖励，$\gamma \in [0,1]$ 为折扣因子
### 2.2 核心符号

Policy Gradient 在 MDP 基础上引入了以下符号：

| 符号                     | 名称    | 含义                                                         |
| ---------------------- | ----- | ---------------------------------------------------------- |
| $\pi_\theta(a \mid s)$ | 参数化策略 | 由 $\theta$ 决定的网络，输入状态 $s$，输出动作 $a$ 的概率                     |
| $\theta$               | 策略参数  | 策略网络的可学习权重                                                 |
| $\tau$                 | 轨迹    | 一局完整交互序列 $(s_0,a_0,r_0,\dots,s_{T-1},a_{T-1},r_{T-1})$     |
| $P(\tau \mid \theta)$  | 轨迹概率  | 在策略 $\pi_\theta$ 下产生轨迹 $\tau$ 的概率                          |
| $R(\tau)$              | 轨迹回报  | 一条轨迹的折扣总奖励 $\sum_{t=0}^{T-1}\gamma^t r_t$                  |
| $G_t$                  | 未来回报  | 从时刻 $t$ 起的折扣累积奖励 $\sum_{k=t}^{T-1}\gamma^{k-t}r_k$         |
| $J(\theta)$            | 目标函数  | 策略的期望回报 $\mathbb{E}_{\tau\sim\pi_\theta}[R(\tau)]$，我们要最大化它 |
| $b(s)$                 | 基线    | 只依赖状态的基线函数，用于减小方差且不引入偏差                                    |
其中 $S$、$A$、$P(s' \mid s,a)$、$R(s,a,s')$、$\gamma$ 的含义与上篇 MDP 五元组一致。
## 2.3 策略梯度推导
### 第一步：写出梯度的积分形式
$$
\nabla_\theta J(\theta) = \int \nabla_\theta P(\tau|\theta)\, R(\tau)\,d\tau
$$
### 第二步：对数导数技巧（Log‑Derivative Trick）

利用恒等式 $\nabla_\theta f = f \cdot \nabla_\theta \log f$：
$$
\nabla_\theta P(\tau|\theta) = P(\tau|\theta) \cdot \nabla_\theta \log P(\tau|\theta)
$$
代入积分，恢复期望形式：
$$
\nabla_\theta J(\theta) = \mathbb{E}_{\tau\sim\pi_\theta}\big[\nabla_\theta \log P(\tau|\theta) \cdot R(\tau)\big]
$$

### 第三步：展开轨迹概率，消去环境项

轨迹概率可以分解为：
$$
P(\tau|\theta) = p(s_0)\prod_{t=0}^{T-1}\pi_\theta(a_t|s_t)\,p(s_{t+1}|s_t,a_t)
$$
取对数：
$$
\log P(\tau|\theta) = \underbrace{\log p(s_0)}_{\text{与}\theta\text{无关}}
+\sum_{t=0}^{T-1}\log \pi_\theta(a_t|s_t)
+\underbrace{\sum_{t=0}^{T-1}\log p(s_{t+1}|s_t,a_t)}_{\text{环境动力学，与}\theta\text{无关}}
$$
对 $\theta$ 求梯度，与 $\theta$ 无关的项全部消失：
$$
\nabla_\theta \log P(\tau|\theta) = \sum_{t=0}^{T-1}\nabla_\theta \log \pi_\theta(a_t|s_t)
$$
> 🎯 关键意义：环境转移概率 $p(s_{t+1}|s_t,a_t)$ 被消掉了——我们不需要知道环境模型，这就是 model‑free 的来源。

### 第四步：代入得到策略梯度公式
$$
\nabla_\theta J(\theta) = \mathbb{E}_{\tau\sim\pi_\theta}\left[\sum_{t=0}^{T-1}\nabla_\theta \log \pi_\theta(a_t|s_t) \cdot R(\tau)\right]
$$
### 第五步：因果律化简
时刻 $t$ 的动作 $a_t$ 不可能影响 $t$ 之前的奖励，因此 $R(\tau)$ 可以替换为从 $t$ 时刻起的未来回报 $G_t$：
$$
\nabla_\theta J(\theta) = \mathbb{E}_{\tau\sim\pi_\theta}\left[\sum_{t=0}^{T-1}\nabla_\theta \log \pi_\theta(a_t|s_t) \cdot G_t\right]
$$
其中：
$$
G_t = \sum_{k=t}^{T-1}\gamma^{k-t} r_k
$$
这就是**策略梯度定理（Policy Gradient Theorem）** 的实用形式。

等价地，也可以写成状态‑动作期望形式：

$$
\nabla_\theta J(\theta) = \mathbb{E}_{s\sim d^\pi,\,a\sim\pi_\theta}\big[\nabla_\theta \log \pi_\theta(a|s) \cdot Q^\pi(s,a)\big]
$$
其中 $d^\pi(s)$ 为策略 $\pi$ 下的状态访问分布。
## 三、REINFORCE 算法

策略梯度公式中有期望 $\mathbb{E}$，实际计算时需要估计它。不同的估计方法催生了不同的算法。
REINFORCE 是最朴素的方案——用蒙特卡洛采样直接估计。
### 3.1 核心思想

| 设计选择 | REINFORCE 的做法 | 影响 |
| ---- | ---- | ---- |
| 回报估计 | 用实际采样回报 $G_t$ 近似 $Q^\pi(s_t,a_t)$ | 无偏，但方差大 |
| 采样策略 | On‑policy：采样和更新用同一个 $\pi_\theta$ | 数据利用率低，但实现简单 |
| 更新时机 | 必须跑完一整个 Episode 才能计算 $G_t$ | 无法在线逐步更新 |
### 3.2 算法流程
#### 1. **初始化：**
- 设定策略网络参数 $\theta$（通常随机初始化神经网络权重）。
- 设定学习率 $\alpha$、折扣因子 $\gamma$。
#### 2. **循环 episodes：**
##### 2.1【采样阶段】
- 让 Agent 使用当前策略 $\pi_\theta$ 与环境进行交互。
- 必须玩完一整局游戏，记录下完整的轨迹：
$$
\tau = (s_0,a_0,r_0,\, s_1,a_1,r_1,\, \dots,\, s_{T-1},a_{T-1},r_{T-1})
$$
##### 2.2【计算回报阶段】
- 为了实现因果律（当前动作只为后续奖励负责），采用从后往前的逆序计算：
1. 初始化最后时刻的回报 $G_T = 0$。
2. 对于时刻 $t = T-1,\,T-2,\, \dots,\,0$：
$$
G_t = r_t + \gamma \cdot G_{t+1}
$$
-（这一步将每一拍的即时奖励 $r$ 转化为评价动作好坏的累积回报 $G$）
##### 2.3【梯度更新阶段】
- 利用策略梯度公式进行梯度上升（增加高分动作的概率）：
$$
\theta \leftarrow \theta + \alpha \cdot \sum_{t=0}^{T-1}\nabla_\theta \log \pi_\theta(a_t|s_t) \cdot G_t
$$
#### 3. **循环结束：**
- 重复上述采样与更新过程，直到策略 $\pi_\theta$ 的表现趋于稳定（收敛）。
### 3.3 直觉理解

REINFORCE 本质上是一个回报加权的最大似然估计：
$$
\underbrace{\max_{\theta}\sum_{t}\log\pi_\theta(a_t|s_t)}_{\text{监督学习：所有样本等权}}
\quad \longrightarrow \quad
\underbrace{\max_{\theta}\sum_{t}G_t\cdot\log\pi_\theta(a_t|s_t)}_{\text{REINFORCE：好样本权重大}}
$$
- $G_t$ 大（好结果）$\rightarrow$ 增大 $\pi_\theta(a_t|s_t)$ $\rightarrow$ 好动作被强化
- $G_t$ 小甚至为负（坏结果）$\rightarrow$ 减小 $\pi_\theta(a_t|s_t)$ $\rightarrow$ 坏动作被抑制
## 四、方差问题与改进方向

REINFORCE 虽然简洁无偏，但致命弱点是方差大。$G_t$ 包含了整条轨迹的随机性，导致梯度估计噪声大、训练不稳定，后续算法的核心主线就是降方差。
### 4.1 引入 Baseline
将 $G_t$ 替换为 $G_t - b(s_t)$，其中 $b(s_t)$ 是一个只依赖状态的基线函数：
$$
\nabla_\theta J(\theta) = \mathbb{E}\big[\nabla_\theta \log \pi_\theta(a_t|s_t) \cdot \big(G_t - b(s_t)\big)\big]
$$
为什么减 baseline 不影响梯度期望（无偏性证明）：
$$
\mathbb{E}_{a\sim\pi_\theta}\big[\nabla_\theta \log \pi_\theta(a|s)\cdot b(s)\big]
= b(s)\cdot \nabla_\theta \underbrace{\sum_{a}\pi_\theta(a|s)}_{=1} = 0
$$
实践中最常用的 baseline 是状态价值函数 $V^\pi(s)$，这样 $G_t - V^\pi(s_t)$ 就近似了优势函数 $A^\pi(s_t,a_t)$。而沿着这个思路继续走——谁来估计 $V^\pi$、怎么估计、怎么保证更新稳定——就引出了后续一系列算法，后续文章会逐一展开。
### 4.2 对比总结：Value‑based vs Policy‑based

| | Value‑based (Q‑Learning) | Policy Gradient (REINFORCE) |
| ---- | ---- | ---- |
| 学什么 | 价值函数 $Q(s,a)$，间接推导策略 | 策略 $\pi_\theta(a \mid s)$，直接优化 |
| 动作空间 | 仅离散 | 离散 + 连续 |
| 策略类型 | 确定性（$\epsilon$-greedy 探索） | 天然随机 |
| 更新频率 | 每步可更新（TD） | 每局才更新（MC） |
| 样本效率 | 较高（Off‑policy + Replay） | 较低（On‑policy，用完即弃） |
| 方差 / 偏差 | 低方差、有偏（bootstrapping） | 高方差、无偏（MC 采样） |
## 五、代码实现（Gymnasium + CartPole）
```python
import torch
import torch.nn as nn
import torch.nn.functional as F
from torch.distributions import Categorical
import gymnasium as gym

# —— 策略网络 ——
class PolicyNet(nn.Module):

def __init__(self, state_dim, action_dim, hidden=128):
    super().__init__.__init__()
    self.fc1 = nn.Linear(state_dim, hidden)
    self.fc2 = nn.Linear(hidden, action_dim)

def forward(self, x):
    return F.softmax(self.fc2(F.relu(self.fc1(x))), dim=-1)

# —— REINFORCE ——
class REINFORCE:
    def __init__(self, state_dim, action_dim, lr=1e-3, gamma=0.99):
        self.net     = PolicyNet(state_dim, action_dim)
        self.optimizer = torch.optim.Adam(self.net.parameters(), lr=lr)
        self.gamma   = gamma
        self.log_probs = []
        self.rewards  = []

    def act(self, state):
        probs  = self.net(torch.tensor(state, dtype=torch.float32))
        dist   = Categorical(probs)
        action = dist.sample()
        self.log_probs.append(dist.log_prob(action))
        return action.item()

    def update(self):
        # ① 从后往前算折扣回报 G_t
        G, returns = 0.0, []
        for r in reversed(self.rewards):
            G = r + self.gamma * G
            returns.insert(0, G)
        returns = torch.tensor(returns)
        returns = (returns - returns.mean()) / (returns.std() + 1e-8) # 归一化（简易baseline）

        # ② 策略梯度: L = -∑ log π(a_t|s_t) · G_t
        loss = sum(-lp * g for lp, g in zip(self.log_probs, returns))

        # ③ 更新 & 清空
        self.optimizer.zero_grad()
        loss.backward()
        self.optimizer.step()
        self.log_probs.clear()
        self.rewards.clear()

# —— 训练 ——
env  = gym.make("CartPole‑v1")
agent = REINFORCE(env.observation_space.shape[0], env.action_space.n)

for ep in range(500):
    state, _ = env.reset()
    ep_reward = 0.0

    while True:
        action = agent.act(state)
        state, reward, term, trunc, _ = env.step(action)
        agent.rewards.append(reward)
        ep_reward += reward
        if term or trunc:
            break

    agent.update()

    if (ep + 1) % 50 == 0:
        print(f"Episode {ep+1:4d} | Reward: {ep_reward:.0f}")
```

# 强化学习：Actor‑Critic、A2C、A3C

#强化学习 #Actor-Critic #A2C #A3C
## 一、承上启下：REINFORCE 的三大痛点

Policy Gradient 系列算法通过直接优化策略 $\pi_\theta(a|s)$ 来最大化累积奖励期望，最基础的实现 REINFORCE 的核心梯度公式为：
$$
\nabla J(\theta) = \mathbb{E}_{\tau\sim\pi_\theta}\left[\sum_{t=0}^{T-1}\nabla_\theta \log\pi_\theta(a_t|s_t)\cdot G_t\right]
$$
其中，
$G_t = \sum_{k=0}^{T-t}\gamma^k r_{t+k}$ 是从时刻 $t$ 起的折扣累积回报
$\gamma \in (0,1]$ 为折扣因子。

尽管思路直觉清晰，REINFORCE 存在三大核心痛点，直接催生了后续算法：

| 痛点 | 根本原因 | 改进方向 |
| ---- | ---- | ---- |
| 方差极大，训练震荡 | 单条轨迹 $G_t$ 受环境随机性影响大 | 引入价值函数作为基准（AC/A2C） |
| 样本效率低，数据一次性消费 | 更新参数后旧数据分布即失效 | 重要性采样（PPO） |
| 更新无约束，易策略崩溃 | 梯度步长不可控，策略可能突变 | 信任域约束（TRPO/PPO 裁剪） |

## 二、础 Actor‑Critic（AC）
### 2.1 核心架构：双模块协同
REINFORCE 的核心瓶颈在于梯度权重 $G_t$ 是完整轨迹的蒙特卡洛估计，延迟高、方差大。
Actor‑Critic（AC）将策略执行与价值评估拆分为两个协同模块，通过引入可学习的价值函数实现在线 TD 更新：

- **Actor（策略网络，参数 $\theta$）**：输入状态 $s_t$，输出动作概率分布 $\pi_\theta(a|s_t)$，以 Critic 的评估信号为驱动，目标是最大化长期累积奖励；
- **Critic（价值网络，参数 $\phi$）**：输入状态 $s_t$，输出状态价值估计 $V_\phi(s_t)$，为 Actor 提供实时、稳定的梯度更新信号。

相较于 REINFORCE 中 $G_t$ 需等完整轨迹结束才能计算，Critic 使得每步都能在线产生评估信号，这是 AC 的核心工程价值。
### 2.2 Critic 的核心学习信号：单步 TD 误差

Critic 的学习依赖单步时序差分（TD）误差 $\delta_t$：
$$
\delta_t = r_t + \gamma V_\phi(s_{t+1}) - V_\phi(s_t)
$$

| 项 | 含义 |
| ---- | ---- |
| $r_t$ | 当前步获得的即时奖励（真实观测值） |
| $\gamma V_\phi(s_{t+1})$ | 下一状态价值的折扣估计（"自举"未来收益） |
| $r_t+\gamma V_\phi(s_{t+1})$ | TD 目标：即时奖励与折扣未来预期之和，是真实价值的单步近似 |
| $V_\phi(s_t)$ | 当前状态的价值估计（已有预期） |
| $\delta_t$ | "实际收益"与"已有预期"之差，即预测误差 |

$\delta_t$ 的符号有明确的策略信号含义：
$$
\begin{aligned}
\delta_t>0 &\Rightarrow \text{实际收益超出预期，当前动作不错，应提升其选取概率} \\
\delta_t<0 &\Rightarrow \text{实际收益不及预期，当前动作较差，应降低其选取概率}
\end{aligned}
$$
**为什么 $\delta_t$ 能替代 $G_t$ 作为 Actor 的更新权重？**
从数学上，$\delta_t$ 是**优势函数** $A(s_t,a_t)=Q(s_t,a_t)-V(s_t)$ 的无偏估计。推导如下（假设 $V_\phi$ 已充分拟合真实价值函数 $V^\pi$）：
$$
\mathbb{E}_{s_{t+1}}\big[\delta_t \,\big|\, s_t,a_t\big]
= r(s_t,a_t)+\gamma\sum_{s'}P(s'|s_t,a_t)V(s')-V(s_t)
= A(s_t,a_t)
$$
其中 $P(s'|s_t,a_t)$ 是环境的状态转移概率。

**优势函数的直觉含义**：$V(s_t)$ 是"基准线"，代表状态 $s_t$ 下所有动作的平均期望价值；$A(s_t,a_t)$ 衡量当前动作相比该状态平均水平的超额收益。以相对量代替绝对量，是 AC 降低方差的根本机制。

> 重要约束：上述推导成立的前提是 $V_\phi$ 已充分准确。当 $V_\phi$ 存在估计误差时，$\delta_t$ 作为 $A_t$ 的估计会引入偏差——这是 AC 系列的固有取舍：以少量偏差换取大幅方差降低。

### 2.3 Actor 与 Critic 的完整更新公式

Critic 更新（最小化 TD 误差的均方值）：
$$
L_{VF}(\phi) = \mathbb{E}_t\big[\delta_t^2\big]
= \mathbb{E}_t\left[\big(r_t+\gamma V_\phi(s_{t+1})-V_\phi(s_t)\big)^2\right]
$$
通过梯度下降最小化该损失，将 $V_\phi(s_t)$ 逐步逼近真实价值 $V^\pi(s_t)$。

> 工程细节：计算 $\nabla_\phi L_{VF}$ 时，通常将 $V_\phi(s_{t+1})$ 视为常量（stop gradient），仅对 $V_\phi(s_t)$ 求偏导。若两端同时求导，TD 自举（bootstrapping）会使训练目标不断漂移，引发不稳定。

Actor 更新（策略梯度，以 $\delta_t$ 为权重）：
$$
\nabla_\theta J(\theta) = \mathbb{E}_t\big[\nabla_\theta \log\pi_\theta(a_t|s_t)\cdot \delta_t\big]
$$
与 REINFORCE 的核心对比：

| 对比维度 | REINFORCE | Actor‑Critic（TD 版） |
| ---- | ---- | ---- |
| 梯度权重 | $G_t$（完整轨迹回报） | $\delta_t$（单步 TD 误差） |
| 是否需要完整轨迹 | 是（等轨迹结束才计算） | 否（每步在线更新） |
| 方差来源 | $G_t$ 范围大，受随机轨迹影响 | $\delta_t$ 衡量相对偏差，范围更小 |
| 偏差 | 低（蒙特卡洛无偏） | 有（$V_\phi$ 不精确时引入偏差） |

## 三、A2C 与 A3C：同步与异步优势 Actor‑Critic
### 3.1 A2C：优势函数的规范引入

基础 AC 中，$\delta_t$ 已隐式近似了优势函数。A2C 的核心贡献是显式定义并规范化优势函数，将其作为核心更新信号。

#### **优势函数的显式定义**
$$
A(s_t,a_t) = Q(s_t,a_t)-V_\phi(s_t)
$$
$V_\phi(s_t)$ 作为"基准线"，消除了状态本身价值对梯度的干扰，使梯度信号仅反映当前动作与平均水平的相对优劣：
- $A_t>0$：当前动作优于该状态平均水平 → 提升其选取概率
- $A_t<0$：当前动作劣于该状态平均水平 → 降低其选取概率

工程近似：维护独立的 Q 网络计算代价较高。由于 $\delta_t$ 是 $A_t$ 的无偏估计，A2C 沿用 TD 误差作为优势的近似：
$$
A_t \approx \delta_t = r_t+\gamma V_\phi(s_{t+1})-V_\phi(s_t)
$$
#### **优势标准化（关键工程技巧）**
在每个训练批次内，对计算出的优势值进行 Z‑score 标准化：
$$
A_t \leftarrow \frac{A_t-\mu_{\text{batch}}}{\sigma_{\text{batch}}+\epsilon},\quad \epsilon \approx 10^{-8}
$$

作用：将批次内优势值统一到均值为 0、标准差为 1 的分布，消除不同任务中奖励尺度差异带来的影响，防止批次中少数极端大值"绑架"整批梯度方向，实践中对训练稳定性有显著正效果。

#### **A2C 核心公式**

Actor 策略梯度：
$$
\nabla J(\theta) = \mathbb{E}_t\big[\nabla_\theta \log\pi_\theta(a_t|s_t)\cdot A_t\big]
$$
Critic 价值损失（使用 MC 回报 $G_t$ 作为高精度监督标签）：
$$
L_{VF}(\phi) = \mathbb{E}_t\big[\big(V_\phi(s_t)-G_t\big)^2\big]
$$
其中 $G_t=\sum_{k=0}^{T-t}\gamma^k r_{t+k}$ 是从时刻 $t$ 开始的实际折扣累积回报。

### 3.2 A3C：异步并行训练

A3C 在 A2C 算法基础上引入多 Worker 异步并行架构，解决单线程训练速度慢的瓶颈。

**核心架构：**
- **全局网络**：共享的 Actor‑Critic 网络，存储并维护全局参数 $\theta,\phi$；
- **Worker 线程（$N$ 个）**：每个 Worker 持有独立的局部参数副本，与独立的环境实例异步交互。

更新流程： Worker 从全局网络拉取参数 → 独立采集 $n$ 步数据并计算梯度 → 无需等待其他 Worker，直接异步推送梯度更新全局网络 → 循环往复。

| 维度 | A3C 的表现 |
| ---- | ---- |
| 训练速度 | 多线程并行，显著优于单线程 |
| 样本多样性 | 不同 Worker 探索不同状态区域，避免局部最优 |
| 梯度一致性 | "过时梯度"：Worker 的局部梯度基于稍旧参数计算，存在理论不一致 |

# 强化学习：从 TRPO 到 PPO

#强化学习 #TRPO #PPO #KL
## 一、TRPO：信任域策略优化（PPO 的前身）
### 1.1 核心背景与 KL 约束
A2C 的策略更新缺少显式的信任域约束：当梯度步长过大时，策略可能剧烈突变，导致新采集数据质量下降，训练过程变得不稳定。TRPO 引入信任域约束，用 KL 散度显式限制新旧策略的分布差异：
$$
D_{KL}(P \parallel Q)=\sum_{x\in\mathcal{X}} P(x)\log\frac{P(x)}{Q(x)},\quad D_{KL}\ge0 \ (\text{非负性})
$$
在策略优化中，取所有状态下新旧策略 KL 散度的期望作为约束：
$$
\bar D_{KL}(\theta_{old}\|\theta)=\mathbb{E}_{s\sim\pi_{\theta_{old}}}\big[D_{KL}(\pi_\theta(\cdot|s)\parallel \pi_{\theta_{old}}(\cdot|s))\big]
$$
### 1.2 TRPO 目标函数
$$
\begin{aligned}
\max_{\theta}\quad &\mathbb{E}_t\left[\frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}\cdot A_t\right] \\
\text{s.t.}\quad &\bar D_{KL}(\theta_{old}\|\theta)\le \delta
\end{aligned}
$$
核心局限：求解该约束优化通常需要 Fisher 向量积、共轭梯度和线搜索等近似步骤，实现复杂、计算开销较大，工程落地成本较高。这正是 PPO 的出发点。
## 二、PPO：近端策略优化

PPO 保留了 TRPO "限制策略更新幅度"的核心理念，用简洁的裁剪（Clip）机制替代复杂 KL 约束，配合重要性采样实现数据复用，兼顾样本效率、训练稳定性与实现简洁性。
### 2.1 重要性采样：打破数据"一次性消费"
#### **核心问题**

策略梯度属于在线策略（On‑Policy）算法：每次参数更新后，旧数据因策略分布改变而失效，必须重新采样。重要性采样（IS）是解决这一问题的标准统计手段。
#### **数学推导**

设目标期望在新策略 $\pi_\theta$ 下定义，但样本来自旧策略 $\pi_{\theta_{old}}$：
$$
\mathbb{E}_{a\sim\pi_\theta}[f(a)]=\sum_{a}\pi_{\theta_{old}}(a)\cdot \underbrace{\frac{\pi_\theta(a)}{\pi_{\theta_{old}}(a)}}_{r_t(\theta)}\cdot f(a)
=\mathbb{E}_{a\sim\pi_{\theta_{old}}}\big[r_t(\theta)\cdot f(a)\big]
$$
定义重要性采样比率：
$$
r_t(\theta)=\frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}
$$
- $r_t(\theta)=1$：新旧策略对该动作概率完全一致；
- $r_t(\theta)>1$：新策略更倾向于选择该动作；
- $r_t(\theta)<1$：新策略更少选择该动作。

通过 $r_t(\theta)$ 加权，旧策略采集的同一批数据可在多个 epoch 中反复使用，彻底打破"一次采集、一次更新"的限制。

应用到策略优化，可得到未加裁剪的代理目标：
$$
L^{IS}(\theta)=\mathbb{E}_t\big[r_t(\theta)\cdot A_t\big]
$$

关键约束：重要性采样要求新旧分布差异不能过大，否则 $r_t(\theta)$ 的方差会爆炸，梯度估计彻底失效。这正是 PPO 引入裁剪操作的根本动机。

### 2.2 PPO‑Clip：裁剪机制替代 KL 约束
#### **核心公式**
$$
L^{CLIP}(\theta)=\mathbb{E}_t\big[\min\big(r_t(\theta)\cdot A_t,\ \text{clip}(r_t(\theta),\,1-\epsilon,\,1+\epsilon)\cdot A_t\big)\big]
$$
- $\epsilon$：裁剪系数，工业界默认 0.2，将 $r_t(\theta)$ 的有效区间限制在 $[0.8,1.2]$；
- $A_t$：优势函数估计值（完整版本通常使用 GAE，详见 2.3 节）。
#### **裁剪操作的含义**
$$
\text{clip}(r_t(\theta),\,1-\epsilon,\,1+\epsilon)=
\begin{cases}
1-\epsilon & r_t(\theta)<1-\epsilon \\
r_t(\theta) & 1-\epsilon \le r_t(\theta)\le 1+\epsilon \\
1+\epsilon & r_t(\theta)>1+\epsilon
\end{cases}
$$
当 $r_t(\theta)\in[1-\epsilon,1+\epsilon]$ 时，裁剪无效，两项相等；当 $r_t(\theta)$ 超出区间时，裁剪项变为常数，其对 $\theta$ 的梯度为零。

#### **min 操作的核心作用：悲观保守下界**

min 始终取两项中的较小值（悲观下界），防止策略越界后依然能持续受益。
##### **情况一：$A_t>0$（当前动作好，应提升其概率）**

| $r_t(\theta)$ 范围 | min 取值 | 梯度 | 效果 |
| ---- | ---- | ---- | ---- |
| $r_t\in[1-\epsilon,1+\epsilon]$ | $r_tA_t$（两项相等） | 有效 | 正常梯度，继续提升动作概率 |
| $r_t>1+\epsilon$ | $(1+\epsilon)A_t$（裁剪项更小） | 截断 | 阻止继续提升（已偏离足够大） |
| $r_t<1-\epsilon$ | $r_tA_t$（未裁剪项更小） | 有效 | 正常梯度（纠正偏离，回调至正确方向） |

##### **情况二：$A_t<0$（当前动作差，应降低其概率）**

| $r_t(\theta)$ 范围 | min 取值 | 梯度 | 效果 |
| ---- | ---- | ---- | ---- |
| $r_t\in[1-\epsilon,1+\epsilon]$ | $r_tA_t$（两项相等） | 有效 | 正常梯度，继续降低动作概率 |
| $r_t>1+\epsilon$ | $r_tA_t$（更负，未裁剪项更小） | 有效 | 正常梯度（纠正偏离，新策略不该增大坏动作概率） |
| $r_t<1-\epsilon$ | $(1-\epsilon)A_t$（裁剪项更负，即更小） | 截断 | 阻止继续压制（已偏离足够大） |

两个关键截断场景总结：
- $A_t>0,\ r_t>1+\epsilon$：好动作的概率已被推得远超旧策略，停止进一步激励；
- $A_t<0,\ r_t<1-\epsilon$：坏动作的概率已被压得远低于旧策略，停止进一步惩罚。

这两个截断共同限制了策略更新幅度：虽然它并不等价于显式的 KL 约束，但在实践中能起到与 TRPO 类似的稳定化作用。

### 2.3 广义优势估计（GAE）

#### **偏差‑方差权衡问题**

| 估计方法 | 公式 | 偏差 | 方差 |
| ---- | ---- | ---- | ---- |
| 单步 TD | $\delta_t = r_t+\gamma V_\phi(s_{t+1})-V_\phi(s_t)$ | 高 | 低 |
| $n$ 步 return | $\sum_{k=0}^{n-1}\gamma^k r_{t+k}+\gamma^n V(s_{t+n})-V(s_t)$ | 中 | 中 |
| MC 回报 | $G_t-V_\phi(s_t)$ | 低 | 高 |
| GAE ($\lambda$) | $\hat A_t^{GAE(\lambda)}$ | 由 $\lambda$ 连续调控 | 由 $\lambda$ 连续调控 |

#### **GAE 公式推导**

定义每步单步 TD 误差：$\delta_t = r_t+\gamma V_\phi(s_{t+1})-V_\phi(s_t)$

GAE 对所有 $k$ 步的 TD 误差进行指数衰减加权求和：
$$
\hat A_t^{GAE(\lambda)}=\sum_{k=0}^{\infty}(\gamma\lambda)^k \delta_{t+k}
=\delta_t+(\gamma\lambda)\delta_{t+1}+(\gamma\lambda)^2\delta_{t+2}+\cdots
$$
权重 $(\gamma\lambda)^k$ 随 $k$ 增大呈指数衰减，使近期 TD 误差权重更高，远期影响逐渐减弱。

#### **边界情况验证**

当 $\lambda=0$（退化为单步 TD 优势，低方差、高偏差）：
$$
\hat A_t^{GAE(0)}=\delta_t = r_t+\gamma V_\phi(s_{t+1})-V_\phi(s_t)
$$
当 $\lambda=1$（退化为 MC 优势，低偏差、高方差）：
$$
\hat A_t^{GAE(1)}=\sum_{k=0}^{T-t-1}\gamma^k \delta_{t+k}
=\sum_{k=0}^{T-t-1}\gamma^k r_{t+k}+\underbrace{\gamma^{T-t}V_\phi(s_T)}_{\approx 0}-V_\phi(s_t)
\approx G_t-V_\phi(s_t)
$$
当 $\lambda=0.95$（工业界默认值）：在偏差与方差之间取折中，经大量实验验证是稳定性最佳的区间。

#### **工程实现：从末端反向递推**
$$
\hat A_T=0,\quad \hat A_t=\delta_t+\gamma\lambda\cdot \hat A_{t+1}
$$
时间复杂度 $O(T)$，工程上高效且易实现。

### 2.4 PPO 总损失函数
$$
L_{total}(\theta,\phi)=\mathbb{E}_t\big[L^{CLIP}(\theta)-c_1L_{VF}(\phi)+c_2S[\pi_\theta](s_t)\big]
$$

#### 逐项解析：

$L^{CLIP}(\theta)$：策略裁剪损失（最大化，取正号）
如 2.2 节所述，这是 PPO 的核心优化目标。代码实现中通常会对其取负值，转写为最小化问题。

$-c_1L_{VF}(\phi)$：价值函数损失（最小化，取负号）
$$
L_{VF}(\phi)=\mathbb{E}_t\big[\big(V_\phi(s_t)-G_t\big)^2\big]
$$
Critic 的均方误差损失，以实际累积回报 $G_t$ 为监督目标，最小化价值估计偏差。总损失中取减号：对总损失做梯度上升时，减去一个最小化目标 $L_{VF}$，等价于驱动 $L_{VF}$ 下降。
系数 $c_1$ 工业界常用默认值为 0.5（OpenAI Baselines 等主流实现）。

$+c_2S[\pi_\theta]$：策略熵正则项（最大化，取正号）
$$
S[\pi_\theta](s_t)=-\sum_{a}\pi_\theta(a|s_t)\log\pi_\theta(a|s_t)
$$
熵衡量策略分布的随机性：熵越大策略越均匀，探索性越强；熵越小策略越确定。加入熵正则项，在训练初期维持充分探索，防止策略过早收敛到次优的确定性策略。系数 $c_2$ 通常取 0.01。
#### **符号对应规则**

| 损失项 | 优化方向 | 总损失中的符号 |
| ---- | ---- | ---- |
| $L^{CLIP}$（策略性能） | 最大化 | $+$ |
| $L_{VF}$（价值误差） | 最小化 | $-$ |
| $S$（策略熵） | 最大化 | $+$ |

总损失采用梯度上升进行优化：最大化目标取正号直接优化，最小化目标取负号（加负号后的梯度上升 $\equiv$ 原式梯度下降）。

### 2.5 网络参数共享
Actor 与 Critic **共享底层特征提取层**（Backbone），仅输出头独立，减少总参数量并提升特征复用效率：
$$
f=\text{Backbone}(s;\theta_{shared}),\quad
\pi_\theta(a|s)=\text{ActorHead}(f;\theta_{actor}),\quad
V_\phi(s)=\text{CriticHead}(f;\phi_{critic})
$$

**梯度流向：**
- $\theta_{shared}$：同时接收来自 $L^{CLIP}$ 和 $L_{VF}$ 的梯度；
- $\theta_{actor}$：仅受 $L^{CLIP}$ 影响；
- $\phi_{critic}$：仅受 $L_{VF}$ 影响。

### 2.6 梯度裁剪
反向传播时，若梯度的 $L_2$ 范数超过阈值 $g_{max}$，等比例缩放至 $g_{max}$：

$$
g\leftarrow
\begin{cases}
g & \|g\|_2 \le g_{max} \\
\displaystyle\frac{g_{max}}{\|g\|_2}\cdot g & \|g\|_2>g_{max}
\end{cases}
$$

工业界通常取 $g_{max}=0.5$。等比缩放保留梯度方向，仅调整幅度，不影响优化方向；与 PPO 裁剪机制共同构成"双重稳定"：前者约束策略分布变化幅度，后者约束参数更新步长。

### 2.7 PPO 完整算法流程

PPO（Actor‑Critic）训练流程：

#### **1. 初始化**
初始化策略网络参数 $\theta$ 与价值网络参数 $\phi$（可共享底层特征），并令：
$$
\theta_{old}\leftarrow \theta
$$

#### **2. 数据采集阶段**
使用旧策略 $\pi_{\theta_{old}}$ 与环境交互，采集 $N$ 步轨迹：
$$
\{(s_t,a_t,r_t,s_{t+1},\text{done}_t)\}
$$

#### **3. 优势与目标回报计算**
单步 TD 误差：
$$
\delta_t = r_t+\gamma V_\phi(s_{t+1})-V_\phi(s_t)
$$

GAE（从后往前递推）：
$$
\hat A_t=\delta_t+\gamma\lambda\hat A_{t+1},\quad \hat A_T=0
$$

目标回报：
$$
G_t=\hat A_t+V_\phi(s_t)
$$

优势标准化：
$$
\hat A_t \leftarrow \frac{\hat A_t-\mu}{\sigma+\epsilon}
$$

#### **4. 多轮参数更新（$K$ 个 epoch）**
重复 $K$ 次，每次都将采样数据打乱后按 mini‑batch 训练：

重要性采样比率：
$$
r_t(\theta)=\frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}
$$

PPO Clip 损失：
$$
L^{CLIP}=\mathbb{E}_t\big[\min\big(r_t(\theta)\hat A_t,\ \text{clip}(r_t(\theta),1-\epsilon,1+\epsilon)\hat A_t\big)\big]
$$

价值函数损失：
$$
L^{VF}=\mathbb{E}_t\left[\big(V_\phi(s_t)-G_t\big)^2\right]
$$

策略熵：
$$
S[\pi_\theta](s_t)
$$

总损失：
$$
L_{total}(\theta,\phi)=L^{CLIP}(\theta)-c_1L^{VF}(\phi)+c_2S[\pi_\theta]
$$

- 反向传播，联合更新 $\theta$ 与 $\phi$
- 梯度裁剪（max norm = 0.5）

#### **5. 同步旧策略**
$$
\theta_{old}\leftarrow \theta
$$

| 超参数 | 含义 | 工业界典型值 |
| ---- | ---- | ---- |
| $\epsilon$ | PPO 裁剪系数 | 0.2 |
| $\gamma$ | 折扣因子 | 0.99 |
| $\lambda$ | GAE 平衡参数 | 0.95 |
| $c_1$ | 价值损失权重系数 | 0.5 |
| $c_2$ | 熵正则系数 | 0.01 |
| $K$ | 每批数据复用 epoch 数 | 4~10 |
| $g_{max}$ | 梯度裁剪阈值 | 0.5 |

## 三、算法演进全景总结
**核心演进逻辑：**
1. REINFORCE 奠定策略梯度基础；
2. AC 引入 Critic，通过自举估计降低方差；
3. A2C/A3C 进一步提升训练效率与稳定性；
4. TRPO 从理论上约束策略更新步幅；
5. PPO 则以更简单的工程实现近似继承了 TRPO 的稳定性优势，通过裁剪机制控制更新幅度、通过重要性采样复用数据、通过 GAE 平衡偏差与方差，因此成为实践中最常用的策略梯度算法之一。

# 元学习：Meta‑Learning 是什么

#元学习 #Meta-Learning #Few-shot #MAML

> Learning to Learn —— 学习如何学习

## 一、为什么需要 Meta‑Learning？

传统机器学习通常假设：训练阶段已经有足够多的当前任务数据，模型只需要在这个任务上不断拟合，最后得到一个表现不错的预测函数。

但很多真实场景并不满足这个假设：

| 问题 | 举例说明 |
| ---- | ---- |
| 新任务数据很少 | 医学影像、冷门类别识别、个性化推荐等场景中，每个新任务都只有少量标注样本 |
| 每个任务都重新训练成本太高 | 新类别、新用户、新环境一出现，就从头训练一个模型，效率很低 |
| 普通模型缺少快速适应能力 | 模型只学会了一个固定任务，而不是学会如何迁移到相似的新任务 |

Meta‑Learning 的目标就是解决这个问题：让模型在大量相关任务中学习“如何学习”，从而在遇到一个全新任务时，只需要少量样本就能快速适配。

一句话说：
> Machine Learning 学习的是 **完成一个任务**；
> Meta‑Learning 学习的是 **如何快速学会新的任务**。

## 二、从 Machine Learning 到 Meta‑Learning
### 2.1 Machine Learning 学什么？

传统机器学习希望学习一个映射函数：
$$
f: X \to Y
$$
例如图像分类：输入图像，输出类别。
```mermaid
flowchart LR
    Image --> Classifier --> Cat/Dog

```
训练目标是找到一个能够完成当前任务的最优函数：
$$
f^*=\arg\min_{f} L(f)
$$

标准流程是：
1. 定义带未知参数的模型函数 $f_\theta$
2. 在当前任务训练集上定义损失函数 $L(\theta)$
3. 优化得到最优参数：
$$
\theta^*=\arg\min_{\theta} L(\theta)
$$
所以普通机器学习学到的是一个**任务内预测函数**。

### 2.2 Meta‑Learning 学什么？

Meta‑Learning 会把视角从“单个样本”提升到“多个任务”。假设每个任务 $\mathcal T_i$ 都有自己的少量数据，元学习希望从大量任务中学到一个通用学习算法 $F_\phi$：
$$
F_\phi: \mathcal D_i^{support} \to f_{\theta_i}
$$
含义是：输入第 $i$ 个任务的少量 Support Set，输出适配该任务的任务特定模型 $f_{\theta_i}$。

所以 Meta‑Learning 不是直接学习某一个最终模型，而是在学习：

> 给我一个新任务的少量样本，我应该如何快速得到一个适合这个任务的模型？

也就是说，Meta‑Learning 扩大了“可学习对象”的范围。

| 过去通常人工设定 | Meta‑Learning 中可以被学习 |
| ---- | ---- |
| 网络参数初始化 | 学一个适合快速微调的初始化，例如 MAML |
| 特征空间 | 学一个让相似类别靠近、不同类别远离的嵌入空间 |
| 优化规则 | 学一套参数更新策略或学习率策略 |
| 模型结构 / 记忆机制 | 学会如何利用任务上下文和历史信息 |

因此：
> Meta‑Learning 学习的是 **Learning Algorithm**，而不只是最终的预测模型。

### 2.3 核心区别

| 维度 | 普通机器学习（Machine Learning） | 元学习（Meta‑Learning） |
| ---- | ---- | ---- |
| 学习目标 | 学习单任务预测函数 $f_\theta$ | 学习跨任务的学习算法 $F_\phi$ |
| 数据单位 | 样本 Sample | 任务 Task / Episode |
| 训练方式 | 在一个任务的数据上优化 | 在大量任务上反复训练和验证适应能力 |
| 损失来源 | 当前任务训练集 | 每个任务适配后的 Query Set |
| 优化对象 | 模型参数 $\theta$ | 元参数 $\phi$，例如初始化、嵌入空间、优化器等 |
| 泛化目标 | 泛化到同一任务的新样本 | 泛化到训练时没见过的新任务 |

## 三、元学习的数据层级

Meta‑Learning 的关键不是只拆训练集和测试集，而是要在**任务层级**和**任务内部**同时划分数据。
### 3.1 跨任务层级

完整元数据集通常被拆成三类任务集合：

| 任务集合 | 作用 |
| ---- | ---- |
| meta‑train tasks | 用大量基础任务训练通用学习算法 $F_\phi$ |
| meta‑validation tasks | 调超参、早停、选择 checkpoint |
| meta‑test tasks | 使用训练阶段从未见过的新任务，最终评估快速适应能力 |

可以理解为：
> Meta Dataset → Meta‑train Tasks / Meta‑validation Tasks / Meta‑test Tasks

注意这里切分的是 **Task**，不是单个样本。

### 3.2 任务内部层级

每个任务 $\mathcal T_i$ 内部又会被拆成两部分：

| 数据集 | 作用 |
| ---- | ---- |
| Support Set | 任务内小训练集，用来快速适配当前任务 |
| Query Set | 同一任务的小测试集，用来计算元损失，评估适配后的泛化效果 |

可以理解为：
> Task / Episode → Support Set（任务内学习） + Query Set（计算元损失）

如果是图像 few‑shot 分类，常见说法是 **N‑way K‑shot**：
- **N‑way**：每个任务包含 $N$ 个类别
- **K‑shot**：每个类别在 Support Set 中只有 $K$ 个标注样本

例如 5‑way 1‑shot 表示：每个 episode 有 5 个类别，每个类别只有 1 张 support 图片。

## 四、训练机制：Inner Loop、Outer Loop 与 Meta Loss

Meta‑Learning 通常包含两个层次的学习：
- Within‑task Learning / **Inner Loop**：在单个任务内部快速适配
- Across‑task Learning / **Outer Loop**：跨多个任务更新元参数

### 4.1 Inner Loop：任务内适配

Inner Loop 发生在单个任务内部：
> Support Set → Task Adaptation → Task‑specific Model

以 MAML 为例，元参数 $\phi$ 可以理解为一个“容易快速适应的初始化”。对任务 $\mathcal T_i$，先在 Support Set 上做一小步或几小步梯度更新：
$$
\theta_i' = \phi - \alpha \nabla_\phi L_{\mathcal T_i}^{support}(\phi)
$$
其中：
- $\phi$：元参数，也就是跨任务共享的初始化
- $\theta_i'$：适配第 $i$ 个任务后的临时参数
- $\alpha$：inner loop 学习率
- $L_{\mathcal T_i}^{support}$：第 $i$ 个任务在 Support Set 上的损失

这一过程称为 **Task Adaptation**。

### 4.2 Outer Loop：跨任务更新

Outer Loop 不直接看 Support Loss，而是看模型适配之后在 Query Set 上表现如何。

单个任务的训练链路是：
> Support Set → Adapt → Task‑specific Model → Query Set → Query Loss

多个任务合在一起就是：
> Task 1/2/3/… 的 Query Loss → 累加为 Meta Loss → 更新元参数 $\phi$

元损失写成：
$$
L_{meta}(\phi)=\sum_{i=1}^{N} L_{\mathcal T_i}^{query}(\theta_i')
$$
优化目标是：
$$
\phi^*=\arg\min_{\phi} L_{meta}(\phi)
$$
用梯度下降更新时：
$$
\phi \leftarrow \phi - \beta \nabla_\phi L_{meta}(\phi)
$$
其中：
- $N$：一个 meta‑batch 中的任务数量
- $L_{\mathcal T_i}^{query}$：第 $i$ 个任务在 Query Set 上的损失
- $\beta$：outer loop 学习率

> 💡 Important
> Meta‑Learning 的关键不是让模型在 Support Set 上记住少量样本，而是让模型经过 Support Set 适配后，能在同任务的 Query Set 上表现好。

### 4.3 Loss 的区别

对比普通机器学习，二者的 Loss 来源不同：

| 方法 | Loss 来源 | 优化目标 |
| ---- | ---- | ---- |
| Machine Learning | Training Data → Prediction → Training Loss | 最小化当前任务训练损失 $L(\theta)$ |
| Meta‑Learning | Support Set → Adapt → Query Set → Query Loss | 最小化多个任务的 Query Loss $L_{meta}(\phi)$ |

如果 $L_{meta}$ 对 $\phi$ 可微，就可以用梯度下降更新；如果不可微，也可以用强化学习、进化算法等方式搜索元参数或元策略。

## 五、为什么 Meta‑Learning 能 Few‑shot？

传统 Machine Learning 面对新任务时，往往需要大量新数据重新训练：
> New Task → 大量数据 → 重新训练

Meta‑Learning 面对新任务时，只需要使用少量 Support Set 做快速适配：
> New Task Support Set → 少量更新 / 度量匹配 / 记忆读取 → Task‑specific Model

### 5.1 快速适配的原因

原因在于：模型已经在大量 meta‑train tasks 中学过任务之间的共性。

这些共性可能表现为：
- 一个容易快速微调的参数初始化
- 一个适合比较样本相似度的特征空间
- 一套有效的更新规则
- 一种利用上下文和记忆的机制

所以 few‑shot 并不是“凭空从一个样本中学会一切”，而是：
> 先从大量相关任务中积累可迁移的学习经验，再用少量新样本完成任务特定适配。

### 5.2 适用边界

这个假设也说明了 Meta‑Learning 的边界：如果 meta‑test 任务和 meta‑train 任务完全不相关，快速适应能力会明显下降。

## 六、Meta‑Learning 的常见算法

### 6.1 算法谱系

不同元学习算法的差别，本质上在于：它们选择让模型学习“学习过程”的哪一部分。

| 算法类别 | 核心思路 | 代表算法 |
| ---- | ---- | ---- |
| Optimization‑based<br>学习如何优化 | 学一个适合快速更新的初始化或优化过程，新任务上做几步梯度更新即可适配 | MAML、Reptile |
| Metric‑based<br>学习嵌入空间 | 学一个特征空间，让同类样本距离近、不同类样本距离远，新样本通过相似度完成分类 | Siamese Networks、Matching Networks、Prototypical Networks、Relation Networks |
| Model‑based<br>学习记忆或更新机制 | 用记忆模块、递归网络或注意力机制，让模型从少量样本中快速写入和读取任务信息 | Memory‑Augmented Neural Networks、Meta Networks、SNAIL |
| Learned Optimizer<br>学习优化器 | 直接学习参数更新规则，让优化器本身成为可训练模型 | LSTM Optimizer、Meta‑learner |

可以把它们粗略理解为：
- MAML：学一个好起点
- Prototypical Networks：学一个好空间
- Learned Optimizer：学一套好更新规则
- Memory‑based 方法：学一种快速记忆和调用任务信息的机制
## 七、总结

> 📝 Summary
> 一句话总结 Meta‑Learning：
>
> Machine Learning 关心“如何把这个任务学好”；
> Meta‑Learning 关心“如何让模型以后学新任务时学得更快”。


# 机器学习与元学习 (Meta‑Learning) 对比

#元学习 #机器学习 #小样本学习 #Meta-Learning
## 1. 传统机器学习（Machine Learning, ML）

传统机器学习面向**单一固定的数据分布**进行建模，利用该分布下大量独立同分布样本训练模型参数。

输入服从单一分布 D 的训练数据，送入学习模型 $f_\theta$（$\theta$为模型参数），经过训练后输出预测结果。

**优化目标：** 在数据分布 D 上学习得到最优模型参数 $\theta^*$，最小化预测损失。

$\theta^*=\arg\min_{\theta}\mathbb E_{(x,y)\sim D}\left[\mathcal L(f_\theta(x),y)\right]$

传统机器学习的核心是**学习模型参数**，模型泛化能力体现在对**同一数据分布下未见过的新样本**做出正确预测。当任务本身发生改变，通常需要重新收集数据、从头训练模型。
## 2. 元学习（Meta‑Learning）

> 核心思想：从多个任务中学习**如何学习**，使模型拿到少量样本就能够快速适应全新任务，也被称为 “学会学习”。

元学习不再以单个样本作为训练单元，而是以==任务 (Task)==为基本单元，在一个任务分布中采样大量不同任务 \(T_1,T_2,...,T_N\)，交由元学习算法 F，元算法学习通用的学习策略、模型初始化方式或者优化规则，最终面对从未见过的新任务，仅依靠少量样本就完成快速适配。
### 任务内部数据划分

每一个任务 \(T_i\) 的数据集拆分为两部分：
1. **支持集（Support Set）**：用于任务内快速微调、适配当前任务；
2. **查询集（Query Set）**：充当该任务的测试集，用来评估任务适配后的效果，用于元层面更新。
### 双层循环训练机制

- **内循环 Inner Loop（任务内适配）**：利用支持集对模型做局部更新，得到该任务专属参数 $\theta_i'$。

$\theta_i' = \text{Update}(\theta, \text{Support Set})$

- **外循环 Outer Loop（跨任务元更新）**：在查询集上计算元损失，利用多个任务的元损失更新元学习算法F。

$\mathcal L_{meta}= \mathcal L(f_{\theta_i'}, \text{Query Set})$

**元学习优化目标：** 训练得到通用学习策略 $F^$ ，使得经过该策略快速适配后的模型，在各个任务查询集上损失最小。

$F^*=\arg\min_{F}\sum_{i=1}^{N}\mathcal L_{T_i}^{query}\left(f_{\theta_i'(F)},\text{Query Set}\right)$

## 3. 机器学习 vs 元学习核心对比

|维度|机器学习（ML）|元学习（Meta‑Learning）|
|---|---|---|
|学习对象|模型参数 \(\theta^*\)|学习策略 / 模型初始化 / 优化器 F|
|数据来源|来自单一数据分布（i.i.d 样本）|来自任务分布下的多个不同任务样本|
|训练方式|直接在训练集上学习参数|内循环（任务内适配）+ 外循环（跨任务元优化）双层循环|
|评估方式|同一分布的测试集上评估|在全新任务的查询集上评估|
|泛化能力|向**同分布新样本**泛化|向**全新未见过的任务**快速适应泛化|
|典型算法|GD、SGD、Adam|MAML、Reptile、ProtoNet、ANIL|

## ✨一句话总结

机器学习是“学参数”**，解决固定任务；元学习是**“学如何学”**，解决少量样本下快速适应新任务的问题。

## 📝通俗理解

- **传统机器学习**：给大量猫狗图片训练识别猫狗；遇到鸟类识别任务，需要重新准备大量鸟类数据从头训练。
- **元学习**：用大量各式各样的小分类任务做训练，让模型学会 “如何只用几张样本就掌握一个新分类”；遇到鸟类识别任务，只需要寥寥几张鸟类样本，就能快速完成识别。

> 关键差异小结
> 1. ML 最小单元是样本；元学习最小单元是任务（Episode）
> 2. ML 直接用训练样本损失更新模型；元学习使用支持集做适配，查询集计算元损失更新元策略
> 3. ML 输出训练完毕的模型；元学习输出一套学习策略，遇到新任务再生成任务专属模型