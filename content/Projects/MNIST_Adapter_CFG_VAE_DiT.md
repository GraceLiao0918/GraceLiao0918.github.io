---
title: MNIST图像条件生成
tags:
  - 扩散模型
  - Adapter
---

# MNIST图像条件生成


> 课程：[Flow Matching and Diffusion Models — 2026 Version](https://diffusion.csail.mit.edu/2026/index.html)
> 主题：MNIST 条件生成、Classifier-Free Guidance（CFG）、Diffusion Transformer（DiT）、VAE 与潜空间生成、Adapter。


## 1.  概述

将二维分布生成扩展到 MNIST 图像。图像是高维张量，实验以 `32 × 32` 灰度图像为例，并逐步构建三种生成方案：

1. **像素空间生成**：直接在 `1 × 32 × 32` 图像空间训练带类别条件的向量场。
2. **潜空间生成**：先训练 VAE，将图像压缩到小尺寸潜变量，再在潜空间训练条件生成模型，最后由 VAE 解码回图像。
3. Adapter微调：在训练好的潜空间生成模型上进行Adapter微调

方案都使用条件 Flow Matching，并用 CFG 在采样时增强类别条件。

```text
MNIST 图像 x、类别 y
        │
        ├── 像素空间：Gaussian probability path → DiT → 生成图像
        │
        └── VAE 编码器 → 潜变量 z → latent DiT → VAE 解码器 → 生成图像
```

## 2. `MNIST_data.py`

In this section, we'll get a feel for MNIST. We'll then experiment with adding noise to MNIST with `ConditionalGaussianProbabilityPath`.
### 2.1 MNIST DataSet Download

```python
self.dataset = datasets.MNIST(  
    root='./data',  
    train=True,  
    download=True,  
    transform=transforms.Compose([  
        transforms.Resize((32, 32)),  
        transforms.ToTensor(),  
        transforms.Normalize((0.1305,), (0.2891,)),  
    ])  
)
```

### 2.2 Sample

- `Sampleable.sample(n)` 从无标签分布采样数据。
- `LabeledSampleable.sample(n)` 返回 `(samples, labels)`。
- `MNISTSampler` 返回图像 `x`（形状 `(b, 1, 32, 32)`）和类别 `y`（形状 `(b,)`）。

```python
def sample(self, num_samples: int) -> Tuple[torch.Tensor, torch.Tensor]:  
    """  
    Args:        - num_samples: the desired number of samples    Returns:        - samples: shape (batch_size, c, h, w)        - labels: shape (batch_size, label_dim)    """    
    if num_samples > len(self.dataset):  
        raise ValueError(f"num_samples exceeds dataset size: {len(self.dataset)}")  
  
    # 数据处理  
    indices = torch.randperm(len(self.dataset))[:num_samples]  
    samples, labels = zip(*[self.dataset[i] for i in indices])  
    samples = torch.stack(samples).to(self.dummy)  
    labels = torch.tensor(labels, dtype=torch.int64).to(self.dummy.device)  
    return samples, labels
```

### 2.3 ConditionalGaussianProbabilityPath

#### 2.3.1 **概率路径 (Probability Path)**

概率路径是在概率测度空间中连接初始噪声分布与目标数据分布的一条光滑轨迹 $(p_t)_{t \in [0, 1]}$，满足边界条件：

$$p_0 = p_{\text{init}} = \mathcal{N}(0, I_d), \quad p_1 = p_{\text{data}}$$

##### **条件概率路径与边缘概率路径**

直接对未知的整体边缘分布 $p_t(x)$ 进行建模极其困难。流匹配引入了基于单个样本 $z \sim p_{\text{data}}$ 的**条件概率路径** $p_t(x \mid z)$：

$$p_0(\cdot \mid z) = p_{\text{init}} = \mathcal{N}(0, I_d), \quad p_1(\cdot \mid z) = \delta_z$$

由全概率公式，全数据空间上的**边缘概率路径**为：

$$p_t(x) = \int p_t(x \mid z) p_{\text{data}}(z)\, dz$$

#### 2.3.2 **高斯条件概率路径 (Gaussian Probability Path)**

选用单调平滑的噪声调度函数 $\alpha_t, \beta_t$，边界满足 $\alpha_0 = 0, \beta_0 = 1$ 与 $\alpha_1 = 1, \beta_1 = 0$：

$$p_t(x \mid z) = \mathcal{N}(x;\, \alpha_t z,\, \beta_t^2 I_d)$$

对应闭式加噪采样过程：

$$z \sim p_{\text{data}}, \quad \epsilon \sim \mathcal{N}(0, I_d) \implies x_t = \alpha_t z + \beta_t \epsilon \sim p_t(x \mid z)$$

其中 `z` 是数据样本或 VAE 潜变量，`t` 是时间，$\epsilon$是简单分布中的噪声。

```python
def sample_conditional_path(self, z: torch.Tensor, t: torch.Tensor) -> torch.Tensor:  
    """  
    Samples from the conditional distribution p_t(x|z)    Args:        - z: b ...        - t: b    Returns:        - x: b ...    
    """    
    alpha_t = self.rearrange_scalar(self.alpha(t)) # (b 1 1 1)  
    beta_t = self.rearrange_scalar(self.beta(t)) # (b 1 1 1)  
    return alpha_t * z + beta_t * torch.randn_like(z)
```

#### 2.3.3 **直线最优传输调度 (CondOT / Rectified Flow)**

在实验中，令 $\alpha_t = t, \, \beta_t = 1 - t$（此时 $\alpha_t' = 1, \beta_t' = -1$）：

$$x_t = t z + (1 - t) \epsilon, \quad u_t^{\text{target}}(x \mid z) = z - \epsilon$$

优化目标简化为极为优雅的均方误差回归：

$$\mathcal{L}_{\text{CondOT}}(\theta) = \mathbb{E}_{t, z, \epsilon} \left[ \| u_t^\theta(t z + (1 - t)\epsilon) - (z - \epsilon) \|^2 \right]$$

```python
class LinearAlpha(Alpha):  
    """  
    Implements alpha_t = t    """  
    def __call__(self, t: torch.Tensor) -> torch.Tensor:  
        """  
        Args:            - t: b        Returns:            - alpha_t: b        """        
        return t  
  
    def dt(self, t: torch.Tensor) -> torch.Tensor:  
        """  
        Evaluates d/dt alpha_t.        Args:            - t: b        Returns:            - d/dt alpha_t b        """        
        return torch.ones_like(t)  
  
class LinearBeta(Beta):  
    """  
    Implements beta_t = 1-t    """    
    def __call__(self, t: torch.Tensor) -> torch.Tensor:  
        """  
        Args:            - t: b        Returns:            - beta_t: b        """        
        return 1-t  
  
    def dt(self, t: torch.Tensor) -> torch.Tensor:  
        """  
        Evaluates d/dt alpha_t.        Args:            - t: b        Returns:            - d/dt alpha_t: b        """        
        return - torch.ones_like(t)
```

#### 2.3.4 **高斯条件向量场（Conditional Vector Field）**

模型学习向量场 $u_t^\theta(x_t \mid y)$，目标是路径对应的条件向量场 $u_t^{\mathrm{ref}}(x_t \mid z)$。

设高斯条件流为 $\psi_t^{\text{target}}(x_0 \mid z) = \alpha_t z + \beta_t x_0$。对时间求导：

$$\frac{d}{dt}\psi_t = \alpha_t' z + \beta_t' x_0$$

利用反解关系 $x_0 = \frac{x - \alpha_t z}{\beta_t}$，代入即可消去初值 $x_0$，得到显式解析的**高斯条件目标向量场**：

$$ u_t^{\text{target}}(x \mid z) = \left(\alpha'(t)-\frac{\beta'(t)}{\beta(t)}\alpha(t)\right)z+\frac{\beta'(t)}{\beta(t)}x  $$


```python
def conditional_vector_field(self, x: torch.Tensor, z: torch.Tensor, t: torch.Tensor) -> torch.Tensor:  
    """  
    Evaluates the conditional vector field u_t(x|z)    Args:        - x: b c h w        - z: b c h w        - t: b    
    Returns:        - conditional_vector_field: conditional vector field (num_samples, c, h, w)   
    """    
    alpha_t = self.rearrange_scalar(self.alpha(t)) # b  
    beta_t = self.rearrange_scalar(self.beta(t)) # b  
    dt_alpha_t = self.rearrange_scalar(self.alpha.dt(t)) # b  
    dt_beta_t = self.rearrange_scalar(self.beta.dt(t)) # b  
  
    return (dt_alpha_t - dt_beta_t / beta_t * alpha_t) * z + dt_beta_t / beta_t * x
```

#### 2.3.5 **分数函数（Score Function）**

对于任意概率密度分布 $q(x)$，其**分数函数 (Score Function)** 定义为对数似然的一阶梯度：

$$s(x) := \nabla_x \log q(x) = \frac{\nabla_x q(x)}{q(x)}$$

几何意义：指向概率密度上升最陡峭的方向。

在高斯条件路径 $p_t(x \mid z) = \mathcal{N}(x; \alpha_t z, \beta_t^2 I)$ 

$$\nabla_x \log p_t(x \mid z) = - \frac{x - \alpha_t z}{\beta_t^2}$$

通过待定系数代数整理，流匹配向量场与分数函数存在精确代数对应关系：

$$u_t^{\text{target}}(x) = a_t \nabla_x \log p_t(x) + b_t x$$

$$a_t = \beta_t^2 \frac{\alpha_t'}{\alpha_t} - \beta_t' \beta_t, \quad b_t = \frac{\alpha_t'}{\alpha_t}$$

```python
def conditional_score(self, x: torch.Tensor, z: torch.Tensor, t: torch.Tensor) -> torch.Tensor:  
    """  
    Evaluates the conditional score of p_t(x|z)    Args:        - x: b ...        - z: b ...        - t: b    
    Returns:        - conditional_score: b ...    """    
    alpha_t = self.rearrange_scalar(self.alpha(t))  
    beta_t = self.rearrange_scalar(self.beta(t))  
    return (z * alpha_t - x) / beta_t ** 2
```

### 2.4 Samples under ConditionalGaussianProbabilityPath

对同一组真实图像，在 t=0,0.25,0.5,0.75,1 分别采样中间状态。随着时间增大，图像权重上升、噪声幅度下降。各时间点独立采样噪声，因此各列并非同一固定噪声路径的连续轨迹。

![[mnist_data_sample.png]]

## 3. CFG_CMM.py

### 3.1 Guidance

条件训练本身已经在学习 $p(x\mid y)$。Guidance 是采样阶段额外改变条件影响强度的手段，它并不是让条件生成“首次成为可能”。模型容量有限、训练不足或类别特征混淆时，适当增强类别方向有时能提高标签一致性；但同时可能偏离原始条件分布并降低覆盖范围。

#### 3.1.1 **朴素条件引导 (Vanilla Guidance)**

将条件上下文（文本/类别）编码向量 $y$ 与时间 $t$、噪声状态 $x$ 一同输入神经网络，直接使用引导流匹配目标进行训练：

$$\mathcal{L}_{\text{CFM}}^{\text{target}}(\theta) = \mathbb{E}_{(z, y) \sim p_{\text{data}},\, t \sim [0, 1],\, \epsilon} \left[ \| u_t^\theta(x_t \mid y) - u_t^{\text{target}}(x_t \mid z) \|^2 \right]$$

- **瓶颈**：在没有额外引导的情况下直接采样，生成的样本往往无法高度贴合提示词（即文本对齐度较弱，模式覆盖范围过广）。

#### 3.1.2 **分类器引导 (Classifier Guidance) 

根据贝叶斯法则：$p_t(x \mid y) = \frac{p_t(x) p_t(y \mid x)}{p_t(y)}$，对 $x$ 取空间梯度：

$$\nabla_x \log p_t(x \mid y) = \nabla_x \log p_t(x) + \nabla_x \log p_t(y \mid x)$$

将高斯路径下的代数转换关系代入，条件目标向量场可以显式分解为：

$$u_t^{\text{target}}(x \mid y) = u_t^{\text{target}}(x) + a_t \nabla_x \log p_t(y \mid x)$$

为了强制要求生成结果遵循条件 $y$，人为引入超参数 $w > 1$（引导尺度 Guidance Scale），对分类器似然梯度项进行线性放大：

$$\tilde{u}_t(x \mid y) = u_t^{\text{target}}(x) + w\, a_t \nabla_x \log p_t(y \mid x)$$

#### 3.1.3 **无分类器引导 (Classifier-Free Guidance)**

由于在含噪数据上训练独立的辨别式分类器 $p_t(y \mid x)$ 成本高昂且在多模态（长文本）下难以提供鲁棒梯度，CFG 采用代数重配方将其消除：

$$\nabla_x \log p_t(y \mid x) = \nabla_x \log p_t(x \mid y) - \nabla_x \log p_t(x)$$

代入放大方程整理得：

$$\tilde{u}_t^w(x \mid y) = (1 - w) u_t^{\text{target}}(x) + w\, u_t^{\text{target}}(x \mid y) = u_t(x) + w \big( u_t(x \mid y) - u_t(x) \big)$$

通过引入空条件标记 $\emptyset$，将无条件生成视作特殊条件分支 $u_t^{\text{target}}(x) \equiv u_t^{\text{target}}(x \mid \emptyset)$：
- 在训练循环中，以一定概率 $\eta$（通常取 $10\% \sim 20\%$）随机将真实提示词 $y$ 替换为空标记 $\emptyset$。
- **推理期外推**：在前向步进中同时估算条件场与无条件场并做线性组合，以牺牲生成多样性为代价，显著提升图像与提示词的结构契合度与图像清晰度。


### 3.2 CFGVectorFieldODE

#### 3.2.1 推理时的引导

同一个模型分别使用真实标签和 null label 预测向量场：

$$
\tilde{u}_t^\theta(x\mid y)
=(1-w)u_t^\theta(x\mid\varnothing)+w u_t^\theta(x\mid y).
$$

- `w=1`：普通有条件预测，没有额外 CFG 放大。
- `w>1`：更强地强调类别条件，通常会提高类别一致性，但过大可能降低多样性或造成失真。
- $\eta$控制训练时 null label 样本的比例；`w` 控制生成时条件引导强度，二者作用阶段不同。

```python
class CFGVectorFieldODE(ODE):  
    def __init__(self, net: ConditionalVectorField, null_label: int, guidance_scale: float = 1.0):  
        self.net = net  
        self.guidance_scale = guidance_scale  
        self.null_label = null_label  
  
    def drift_coefficient(self, x: torch.Tensor, t: torch.Tensor, y: torch.Tensor) -> torch.Tensor:  
        """  
        Args:        - x: b ...        - t: b        - y: b        """        
        guided_vector_field = self.net(x, t, y)  
        unguided_y = torch.ones_like(y) * self.null_label  
        unguided_vector_field = self.net(x, t, unguided_y)  
        return (1 - self.guidance_scale) * unguided_vector_field + self.guidance_scale * guided_vector_field
```

### 3.3 CFGTrainer

Training for Classifier-Free Guidance

#### 3.3.1 **条件训练目标**

对每个 batch：
1. 从 MNIST 采样图像 `z` 和标签 `y`。
2. 以概率 $\eta$将标签替换为 null label。MNIST 有 10 个类别，因此常用 `null_label=10`，类别 embedding 需要 11 个索引。
3. 采样时间 `t` 和路径样本 `x_t`。
4. 让模型预测的向量场接近条件路径的参考向量场：

$$
\mathcal{L}_{\mathrm{CFM}}
= \mathbb{E}\left[
\left\|u_t^\theta(x_t\mid y)-u_t^{\mathrm{ref}}(x_t\mid z)\right\|_2^2
\right].
$$

这里的 $\eta$是**训练时丢弃标签的概率**，不是采样时的 guidance scale。

$$ \widehat{\mathcal L} = \frac1B\sum_{i=1}^{B} \sum_{j=1}^{D} \left( u_{\theta,ij}-u^{\mathrm{ref}}_{ij} \right)^2. $$

```python
class CFGTrainer(Trainer):  
    def __init__(self, path: GaussianConditionalProbabilityPath, eta: float, null_label: int, eps: float = 0.001, **kwargs):  
        assert eta > 0 and eta < 1  
        super().__init__(**kwargs)  
        self.eta = eta  
        self.eps = eps  
        self.path = path  
        self.null_label = null_label  
  
    def get_train_loss(self, batch_size: int) -> torch.Tensor:  
        # Step 1: Sample z,y from p_data  
        # raise NotImplementedError("Fill me in!")       
        z, y = self.path.p_data.sample(batch_size)  
  
        # Step 2: Set each label to 10 (i.e., null) with probability eta  
        # raise NotImplementedError("Fill me in!")        
        mask = torch.rand(batch_size, device=y.device) < self.eta  
        y = torch.where(mask, torch.full_like(y, self.null_label), y)  
  
        # Step 3: Sample t and x  
        # raise NotImplementedError("Fill me in!")        
        t = torch.rand(batch_size, device=z.device, dtype=z.dtype) * (1 - self.eps)  
        x = self.path.sample_conditional_path(z, t)  
  
        # Step 4: Regress and output loss  
        # raise NotImplementedError("Fill me in!")        
        pred = self.model(x, t, y)  
        target = self.path.conditional_vector_field(x, z, t)  
        return ((pred - target).reshape(batch_size, -1) ** 2).sum(dim=1).mean()
```


### 3.6 MLPConditionalVectorField

To sanity check our implementation of `CFGTrainer`, we'll train a simple MLP-based model to sample conditionally from a Gaussian mixture. First, let's implement the class `MLPConditionalVectorField`.

`MLPConditionalVectorField` 用于在 GMM 中**学习条件速度场**。它回答的问题是：已知当前点的位置$x_t$、时间t和目标类别y，这个点此刻应该往哪个方向移动、移动多快？

训练时，我们从 GMM 中抽到一个具体的目标点 z和它的标签 y，再抽取高斯噪声 $\epsilon$，构造线性概率路径：

$$ x_t=tz+(1-t)\epsilon, \qquad v_{\mathrm{ref}}=z-\epsilon. $$

因为 z 和 $\epsilon$ 都已知，所以可以算出参考速度。生成时，只指定类别 y，并没有预先选定目标点 z，因此不能直接用 $z-\epsilon$ 更新位置。训练好的 MLP 根据当前状态给出速度预测：

$$\widehat v=u_\theta(x_t,t,y). $$

同一类别中也有许多可能的目标点。平方误差训练让模型在给定当前状态、时间和标签时，拟合与这些输入相容的训练路径的**条件平均速度**，而不需要猜出某个具体终点。


类别标签先通过嵌入表映射为可学习向量 $e_y$，再和坐标、时间拼接：

$$ h_0=[x_t;\,e_y;\,t]\in\mathbb R^5, \qquad u_\theta(x_t,t,y)=\operatorname{MLP}(h_0). $$

MLP 的维度是 $5\rightarrow256\rightarrow256\rightarrow2$，两个隐藏层使用 SiLU，输出层不加激活，使两个速度分量都能取正值或负值。

```python
class MLPConditionalVectorField(ConditionalVectorField):  
  def __init__(  
      self,  
      dim: int,  
      hidden_dim: int,  
      class_dim: int,  
      num_classes: int  
    ):  
    super().__init__()  
    self.mlp = MLP([dim + class_dim + 1, hidden_dim, hidden_dim, dim])  
    self.class_embedding = nn.Embedding(num_classes + 1, class_dim)  
  
  def forward(self, x: torch.Tensor, t: torch.Tensor, y: torch.Tensor):  
      """  
      Args:      - x: b d      - t: b      - y: b      Returns:      - u_t^theta(x|y): (b, c, h, w)      """      
      # raise NotImplementedError("Fill me in!")      
      # 保存原始shape  
      b, *rest, d = x.shape  
      x_flat = x.reshape(b, d)   # 压成 [B,2]      y_embed = self.class_embedding(y)  
      t = t.unsqueeze(-1)  
      inputs = torch.cat([x, y_embed, t], dim=-1)  
      return self.mlp(inputs)
```


### 3.7 GMM
Combining `CFGTrainer`, `CFGVectorFieldODE`, and `MLPConditionalVectorField` to sample from a Gaussian mixture (`GMM`) instance.

高斯混合模型GMM，即 Gaussian Mixture Model，使用多个高斯分量组成复杂分布：

$$ p(z)=\sum_{k=0}^{K-1}\pi_k \mathcal N(z;\mu_k,\Sigma_k) $$

其中：

$$ \pi_k\ge0, \qquad \sum_k\pi_k=1. $$

它是典型的多模态分布：每个分量可以形成一个样本聚集区域。这里的“模态”指概率分布中的聚集结构，不是图像、文本等输入模态。

加入分量标签：

$$ y\sim\operatorname{Categorical}(\pi), \qquad z\mid y=k\sim\mathcal N(\mu_k,\Sigma_k). $$

因此，GMM 天然提供条件生成需要的标签。

#### **Initialize GMM**

- 三个分量的位置

```python
angles = [0, 2 * math.pi / 3, 4 * math.pi / 3]  
means = 2 * torch.tensor([[math.cos(a), math.sin(a)] for a in angles])
```

$$ \mu_0=(2,0), \mu_1=(-1,\sqrt3), \mu_2=(-1,-\sqrt3). $$

它们位于半径为 2 的圆上，相邻角度为 $120^\circ$。

混合权重相同：

$$ \pi_0=\pi_1=\pi_2=\frac13. $$

- `covariances` 实际按标准差使用

```python
covs = torch.tensor([0.2, 0.2, 0.2])
```

但采样代码为：
```
samples[labels == idx] = (
    torch.randn_like(samples[labels == idx])
    * self.covariances[idx]
    + self.means[idx]
)
```

即：

$$ z=\mu_k+0.2\epsilon, \qquad \epsilon\sim\mathcal N(0,I_2). $$

所以实际协方差是：

$$ \Sigma_k=0.2^2I_2=0.04I_2 $$


虽然变量名叫 `covariances`，其在当前实现中实际充当标准差尺度，不能写成协方差为 0.2I。

- 标签采样

```python
weights = torch.tensor([1 / 3, 1 / 3, 1 / 3])
```

```python
labels = torch.multinomial(
    self.weights.cpu(),
    num_samples=num_samples,
    replacement=True,
)
```

这是按权重独立抽取类别。因此，一个批次中三个类别的数量只是期望上相近，并不保证恰好各占三分之一。随后代码对每类分别采样高斯点。

#### **Initialize probability path**


```python
path = GaussianConditionalProbabilityPath(  
    p_data=gmm,  
    p_simple_shape=[2],  
    alpha=LinearAlpha(),  
    beta=LinearBeta()  
).to(device)  
vector_field = MLPConditionalVectorField(  
    dim=2,  
    hidden_dim=256,  
    class_dim=2,  
    num_classes=3  
).to(device)
```

#### **Train vector field**


```python
trainer = CFGTrainer(  
    path=path,  
    eta=0.25,  
    null_label=3,  
)  
losses, steps = trainer.train(model=vector_field, num_steps=3000, lr=1e-3, batch_size=250)  
plt.plot(steps, losses)  
plt.xlabel("Step")  
plt.ylabel("Loss")  
# plt.show()  
plt.savefig('./result/result_total/CFG_GMM_loss.png', dpi=120, bbox_inches="tight")  
plt.close()
```

#### **Visualize Results**

|                          | 内容           | 样本数          | 主要检查       |
| ------------------------ | ------------ | ------------ | ---------- |
| Target                   | 从真实 GMM 直接采样 | 250          | 目标分布位置与形状  |
| Conditioned on each mode | 分别指定类别 0、1、2 | 每类 250，共 750 | 标签是否控制生成区域 |
| Unconditioned            | 全部使用空标签 3    | 750          | 是否覆盖三个分量   |
Unconditioned中，虽然也使用 `CFGVectorFieldODE`，但实际标签为空标签，两分支相同，因此得到无条件速度场。

- guidance_strength = 1.0

![[CFG_GMM_samples.png]]

- guidance_strength = 3.0

![[CFG_GMM_samples_3.png]]

- guidance_strength = 5.0

![[CFG_GMM_samples_5.png]]

- guidance_strength = 10.0

![[CFG_GMM_samples_10 1.png]]

## 4. DiT.py


### 4.1 条件特征嵌入机制 (Embeddings)

1. **时间编码 (Time Embedding)**：标量 $t$ 通过正余弦高频傅里叶特征投影为连续向量，确保网络敏锐感知去噪所处的动力学阶段：

$$\text{TimeEmb}(t) = \sqrt{\frac{2}{d}} \big[ \cos(2\pi \omega_1 t), \dots, \sin(2\pi \omega_{d/2} t) \big]^T$$

2. **文本特征编码 (Text Conditioning)**：
   - **全局对齐**：使用预训练 CLIP 模型获取单一全局语义嵌入向量 $y_{\text{CLIP}}$。
   - **细粒度对齐**：使用 T5 / ByT5 / UL2 等大型语言模型，保留由 Token 序列构成的特征矩阵，用于细粒度交叉注意力交互。

### 4.2 U-Net 到 Diffusion Transformer (DiT)
#### 4.2.1 传统 U-Net 局限

U-Net 依赖下采样-上采样卷积层与跨层跳跃连接（Skip Connections）。当参数扩展到数十亿级别以及需要同时建模多模态长上下文时，局部卷积归纳偏置会限制其泛化能力与吞吐速度。
#### 4.2.2 DiT 核心 (adaLN-Zero 与 Cross-Attention)
DiT 将含噪图像切分为 Patch 并展平成一维序列（Tokenization），利用标准 Transformer 堆叠。
- **自适应层归一化 (adaLN-Zero)**：时间与全局条件向量通过轻量 MLP 回归出缩放/平移系数 $(\gamma, \beta)$ 及门控权重 $\alpha$：

  $$\text{adaLN}(x) = (1 + \gamma) \odot \text{LayerNorm}(x) + \beta$$

- **多模态交叉注意力 (MM-DiT)**：如 Stable Diffusion 3 所示，图像 Token 与文本 Token 分别享有独立的注意力参数映射，但在自注意力层中互相混合计算注意力权重矩阵，实现双向模态特征的深度对齐。
```
图像特征流: [Patch 序列] ---> LayerNorm(adaLN) ---> Q_img \
                                                           (混合注意力) ---> 输出更新
文本特征流: [Text 序列]  ---> LayerNorm(adaLN) ---> K,V_txt /
```

### 4.3 Fourier 时间编码

将标量时间 `t` 编码为正弦、余弦特征：

$$
t^{\mathrm{emb}} =
[\sin(2\pi w_i t),\,\cos(2\pi w_i t)]_{i=1}^{d/2}.
$$

这样网络能以多种频率感知时间位置，而不只是接收单一标量。实现时若 `t` 形状是 `(b, 1, 1, 1)`，传入编码器前需整理成 `(b,)`。

```python
class FourierEncoder(nn.Module):  
    """  
    Based on https://github.com/lucidrains/denoising-diffusion-pytorch/blob/main/denoising_diffusion_pytorch/karras_unet.py#L183    """    def __init__(self, dim: int):  
        super().__init__()  
        assert dim % 2 == 0  
        self.half_dim = dim // 2  
        self.weights = nn.Parameter(torch.randn(1, self.half_dim))  
  
    def forward(self, t: torch.Tensor) -> torch.Tensor:  
        """  
        Args:        - t: b        Returns:        - embeddings: b d        
        """       
        # Step 1: compute frequencies f_i = 2 * pi * w_i * t        
        frequencies = 2 * torch.pi * self.weights * t.unsqueeze(-1)  
  
        # Step 2: compute sin(f_i) and cos(f_i)  
        sin_features = torch.sin(frequencies)  
        cos_features = torch.cos(frequencies)  
  
        # Step 3: Concatenate and return  
        return torch.cat([sin_features, cos_features], dim=-1)
```

### 4.4 Patchifier图像块----向量

输入形状为 $B\times D\times H\times W$。使用核大小与步长都为 $p$ 的卷积：

$$T_{b,n}=W_p\operatorname{vec}(X_{b,\mathrm{patch}\ n})+b_p.$$

输出为 $B\times N\times d$，其中 $N=(H/p)(W/p)$。Token 是网络处理的特征向量，不是像素标签。

```python
class Patchifier(nn.Module):  
  def __init__(self, img_size: int, patch_size: int, c_in: int, dim: int):  
    super().__init__()  
    assert img_size % patch_size == 0, "Image size must be divisible by patch size"  
  
    # raise NotImplementedError("Fill me in!")  
    self.proj = nn.Conv2d(  
    in_channels=c_in,  
    out_channels=dim,  
    kernel_size=patch_size,  
    stride=patch_size,  
    )  
  
  def forward(self, x: torch.Tensor) -> torch.Tensor:  
    """  
    Args:    - x: (bs, 1, img_size, img_size)    Returns:    - x: (bs, 1, img_size, img_size)    """    
    # raise NotImplementedError("Fill me in!")    
    # x: (B, c_in, img_size, img_size)    
    x = self.proj(x)          
    # (B, dim, H/p, W/p)  
    B, D, Hp, Wp = x.shape  
    x = x.flatten(2)          # (B, D, Hp*Wp)  
    x = x.transpose(1, 2)     # (B, Hp*Wp, D)  
    return x
```


### 4.5 多头注意力 MHA

对 token 表示 $H$ 计算查询、键和值：

$$Q=HW_Q,\quad K=HW_K,\quad V=HW_V.$$

查询与键决定关注关系，值提供被汇总的信息。每个头：

$$ A = \operatorname{softmax}(QK^\top/\sqrt{d_h}),\qquad O=AV. $$

除以 $\sqrt{d_h}$ 是为了控制点积随维度增长而扩大的尺度，避免 softmax 过早进入极端饱和状态。softmax 对每个查询对应的键维度归一化，使一行权重和为 1。

多个头允许使用不同投影关系，再拼接并线性映射：

$$\operatorname{MHA}(H)=\operatorname{Concat}(O_1,\ldots,O_h)W_O.$$

实验中，$d=256,h=8,d_h=32$。

```python
class MHA(nn.Module):  
  """  
  Multi-headed self-attention  """  def __init__(self, dim: int, heads: int):  
    super().__init__()  
    assert dim % heads == 0  
  
    # raise NotImplementedError("Fill me in!")  
    self.heads = heads  
    self.head_dim = dim // heads  
    self.qkv = nn.Linear(dim, 3 * dim)  
    self.proj = nn.Linear(dim, dim)  
  
  def forward(self, x: torch.Tensor) -> torch.Tensor:  
    """  
    Args:    - x: b n d    Returns:    - x: b n d    
    """    
    b, n, d = x.shape  
    # Compute queries, keys, and values  
    qkv = self.qkv(x).reshape(b, n, 3, self.heads, self.head_dim)  
    q, k, v = qkv.permute(2, 0, 3, 1, 4).unbind(0)  
  
    # Fold head into batch dimension    
    q = q.reshape(b * self.heads, n, self.head_dim)  
    k = k.reshape(b * self.heads, n, self.head_dim)  
    v = v.reshape(b * self.heads, n, self.head_dim)  
  
    # Compute scaled dot-product attention.    
    attn = torch.softmax(  
        (q @ k.transpose(-2, -1)) / (self.head_dim ** 0.5),  
        dim=-1,  
    )  
    # Combine attention weights with values.  
    out = attn @ v  
    # Unfold heads and combine their features.  
    out = out.reshape(b, self.heads, n, self.head_dim)  
    out = out.transpose(1, 2).reshape(b, n, d)  
  
    return self.proj(out)
```

### 4.6 DiffusionTransformerLayer

#### 4.6.1 **LayerNorm、残差与 SiLU**

LayerNorm 对每个 token 的特征维归一化：

$$\operatorname{LN}(h)=\frac{h-\mu(h)}{\sqrt{\sigma^2(h)+\delta}},$$

其中 $\delta$ 防止除零，具体模块还可能带可学习仿射参数。DiT 层内使用 `elementwise_affine=False`，因为缩放与偏移由条件调制提供。

残差连接 $h+F(h)$ 让网络学习相对输入的修正，而不必每层重新建立完整表示。

激活函数 SiLU 为：

$$\operatorname{SiLU}(a)=a\operatorname{sigmoid}(a).$$

它为网络引入平滑非线性。前馈网络逐 token 运行，维度为 $256\to1024\to256$；注意力负责 token 间交互，前馈网络主要负责各 token 的通道变换。

#### 4.6.2 **AdaLN-Zero**

AdaLN 是 Adaptive Layer Normalization，即根据条件调整归一化后的特征。当前产生六组向量：

$$[b_a,s_a,g_a,b_f,s_f,g_f]=\operatorname{Linear}(\operatorname{SiLU}(c)).$$

注意力分支：

$$H'=H+g_a\odot\operatorname{MHA}((1+s_a)\odot\operatorname{LN}(H)+b_a). $$

前馈分支：

$$H''=H'+g_f\odot\operatorname{FFN}((1+s_f)\odot\operatorname{LN}(H')+b_f).$$


- $b$ 平移特征，$s$ 缩放特征，$g$ 控制残差分支贡献。它们形状为 $B\times d$，沿 token 维广播。

“Zero”指调制线性层零初始化，使初始门控为零，整个 Transformer 层初始为恒等映射。这样可以从较稳定的残差状态开始优化。它不表示最终输出头或整个网络预测一定为零。

```python
class DiffusionTransformerLayer(nn.Module):  
  def __init__(  
      self,  
      dim: int,  
      heads: int,  
  ):  
    """  
    Args:    - n_tokens: sequence length (for sake of positional embeddings)    - dim: dimension of hidden layers    - heads: number of attention heads    
    """    
    super().__init__()  

    # AdaLN supplies the scale and shift, so LayerNorm needs no affine parameters.    
    self.norm1 = nn.LayerNorm(dim, elementwise_affine=False)  
    self.norm2 = nn.LayerNorm(dim, elementwise_affine=False)  
  
    # Produce shift, scale, and gate values for both residual blocks.  
    self.adaLN_modulation = nn.Sequential(  
        nn.SiLU(),  
        nn.Linear(dim, 6 * dim),  
    )  
  
    # Zero initialization makes the residual blocks initially pass x through.  
    nn.init.zeros_(self.adaLN_modulation[-1].weight)  
    nn.init.zeros_(self.adaLN_modulation[-1].bias)  
  
    self.attn = MHA(dim, heads)  
    self.ff = MLP([dim, 4 * dim, dim])  
  
  def forward(self, x: torch.Tensor, c: torch.Tensor) -> torch.Tensor:  
    """  
    Args:    - x: b n d    - c: b d    Returns:    - x: b n d    
    """    
    # Feedforward + residual connection
    shift_attn, scale_attn, gate_attn, shift_ff, scale_ff, gate_ff = (  
        self.adaLN_modulation(c).chunk(6, dim=-1)  
    )  
  
    # Modulate normalized tokens, then apply attention with a gated residual.  
    x = x + gate_attn.unsqueeze(1) * self.attn(  
        self.norm1(x) * (1 + scale_attn.unsqueeze(1))  
        + shift_attn.unsqueeze(1)  
    )  
  
    # Apply the second modulated, gated residual block.  
    x = x + gate_ff.unsqueeze(1) * self.ff(  
        self.norm2(x) * (1 + scale_ff.unsqueeze(1))  
        + shift_ff.unsqueeze(1)  
    )  
  
    return x
```


### 4.7 DiffusionTransformer

#### 4.7.1 **位置编码**

纯注意力不天然知道 token 在二维图像中的位置。加入可学习张量：

$$H_0=T+P,\qquad P\in\mathbb R^{1\times N\times d}.$$

固定 token 顺序与位置编码共同告诉模型相对布局。当前实现是固定长度的可学习位置编码，不能默认支持任意输入分辨率。


```python
class DiffusionTransformer(nn.Module):  
  def __init__(  
      self,  
      depth: int,  
      n_tokens: int,  
      dim: int,  
      **layer_kwargs,  
  ):  
    """  
    Args:    - n_tokens: sequence length (for sake of positional embeddings)    - dim: dimension of hidden layers    - heads: number of attention heads    - depth: number of layers    
    """    
    super().__init__()  
    # raise NotImplementedError("Fill me in!")  
    self.positional_embedding = nn.Parameter(torch.randn(1, n_tokens, dim))  
    self.layers = nn.ModuleList([  
        DiffusionTransformerLayer(dim=dim, **layer_kwargs)  
        for _ in range(depth)  
    ])  
  
  
  def forward(self, x: torch.Tensor, c: torch.Tensor) -> torch.Tensor:  
    """  
    Args:    - x: b n d    - c: b d    Returns:    - x: b n d    """    # raise NotImplementedError("Fill me in!")    x = x + self.positional_embedding  
    for layer in self.layers:  
      x = layer(x, c)  
    return x
```

### 4.8 Depatchifier
输出过程为：

$$B\times N\times d\to B\times N\times(fp^2)\to B\times f\times H\times W\to B\times C\times H\times W.$$

依次对应 LayerNorm、MLP、重排和 $3\times3$ 卷积。`final_dim=10` 中的 10 是中间通道数，不是类别输出。最终每个元素是速度分量，不是类别概率，也不是必须位于 $[0,1]$ 的灰度值。

```python
class Depatchifier(nn.Module):  
  def __init__(self, img_size: int, patch_size: int, dim: int, final_dim: int, c_out: int):  
    super().__init__()  
    self.patch_size = patch_size  
    assert img_size % patch_size == 0, "Image size must be divisible by patch size"  
  
    # raise NotImplementedError("Fill me in!")  
    grid_size = img_size // patch_size  
    self.norm = nn.LayerNorm(dim)  
    self.mlp = MLP([dim, 4 * dim, final_dim * patch_size * patch_size])  
    self.rearrange = Rearrange(  
        "b (h w) (f p1 p2) -> b f (h p1) (w p2)",  
        h=grid_size,  
        w=grid_size,  
        f=final_dim,  
        p1=patch_size,  
        p2=patch_size,  
    )  
    self.conv = nn.Conv2d(final_dim, c_out, kernel_size=3, padding=1)  
  
  def forward(self, x: torch.Tensor) -> torch.Tensor:  
    """  
    Args:    - x: b n d    Returns:    - x: b 1 32 32    """    # raise NotImplementedError("Fill me in!")    x = self.norm(x)  
    x = self.mlp(x)  
    x = self.rearrange(x)  
    return self.conv(x)
```

### 4.9 DiffusionTransformerFlowModel

每层包含注意力和前馈网络两个残差分支。条件向量由时间 embedding 与类别 embedding 相加得到，再通过 AdaLN 调制分支：

$$
\operatorname{modulate}(h,\gamma,\beta)
=h(1+\gamma)+\beta.
$$

每个分支还有门控值控制残差更新。AdaLN 最后一层零初始化时，初始门控和调制量为零，使层在训练初期接近恒等映射，有助于稳定训练。
- `scale` (缩放，对应公式的 $\gamma$)
- `shift` (平移，对应公式的 $\beta$)
- `gate` (门控，对应公式的 $\alpha$)

$$
x_{out} = x + \alpha_1 \cdot \text{Attention}\big( \text{LayerNorm}(x) \cdot (1 + \gamma_1) + \beta_1 \big)
$$

```text
图像 x ─→ Patchifier ─→ 加位置编码 ─→ 多层 DiT ─→ Depatchifier ─→ 向量场输出
时间 t ─→ Fourier Encoder ─┐
类别 y ─→ Embedding ───────┴→ 条件向量 c
```

![[Pasted image 20261106212711.png]]


```python
class DiffusionTransformerFlowModel(ConditionalVectorField):  
  def __init__(  
      self,  
      img_size: int = 32,  
      patch_size: int = 8,  
      num_layers: int = 12,  
      c: int = 1,  
      dim: int = 256,  
      heads: int = 4,  
      final_dim: int = 10,  
      n_classes: int = 11,  
    ):  
      super().__init__()  

      # Encode time to dim features; embed the class label to dim features.  
      self.time_embedder = FourierEncoder(dim)  
      self.y_embedder = nn.Embedding(n_classes, dim)  
  
      # Construct patchifier Convert image to a sequence of patch tokens.  
      self.patchifier = Patchifier(  
          img_size=img_size,  
          patch_size=patch_size,  
          c_in=c,  
          dim=dim,  
      )  
  
      #  Construct DiT  Process patch tokens conditioned on time and class.  
      n_tokens = (img_size // patch_size) ** 2  
      self.dit = DiffusionTransformer(  
          depth=num_layers,  
          n_tokens=n_tokens,  
          dim=dim,  
          heads=heads,  
      )  
  
      # Construct de-patchifier Convert patch tokens back into an image.  
      self.depatchifier = Depatchifier(  
          img_size=img_size,  
          patch_size=patch_size,  
          dim=dim,  
          final_dim=final_dim,  
          c_out=c,  
      )  
  
  def forward(self, x: torch.Tensor, t: torch.Tensor, y: torch.Tensor) -> torch.Tensor:  
    """  
    Args:    - x: b 1 32 32    - t: b 1 1 1    - c: b 1 1 1    Returns:    - u_t^theta(x|y): b 1 32 32    
    """    
    # FourierEncoder expects t with shape (batch,).    
    # 1. Embed time and y
    t = t.reshape(t.shape[0])  
    c = self.time_embedder(t) + self.y_embedder(y)  
    # 2. Patchify
    x = self.patchifier(x)  
    # 3. Pass through DiT 
    x = self.dit(x, c)  
    # 4. Depatchify 
    x = self.depatchifier(x)  
    # return x  
    return x
```


### 4.10 Training DiT

CFG 的核心逻辑：

$$v_{cfg} = (1+\eta)\cdot v_{cond} - \eta\cdot v_{null}$$

这里 `guidance_scale` 就是 CFG 权重（$\eta$）：

- **w = 1.0：引导最弱**-->自由度最高，多样性最强，但有不少模糊、畸形、分不清的坏样本。模型没被强条件约束，更容易跑偏。

- **w = 3.0：引导适中，效果最好**-->数字轮廓清晰，每一行对应同一个类别，结构规整，模糊、崩坏的样本明显变少，是 CFG 最常用的区间。

- **w = 5.0：引导很强**-->数字更锐利、形状更标准；但是代价来了：**多样性下降**，同一个类别的数字长得越来越像，部分样本边缘容易过曝、细节僵硬。如果权重继续拉高，会出现模式崩塌、重复、失真。
![[step_ 5000_output.png]]
```python
def train_dit_cfg():  
    # Initialize our sampler  
    sampler = MNISTSampler().to(device)  
  
    # Initialize probability path  
    path = GaussianConditionalProbabilityPath(  
        # p_data = MNISTSampler(),  
        p_data = sampler,  
        p_simple_shape = [1, 32, 32],  
        alpha = LinearAlpha(),  
        beta = LinearBeta()  
    ).to(device)  
  
    # Initialize model  
    dit = DiffusionTransformerFlowModel(  
        img_size = 32,  
        patch_size = 4,  
        num_layers = 8,  
        c = 1,  
        dim = 256,  
        heads = 8,  
        final_dim = 10,  
        n_classes = 11,  
    ).to(device)  
  
    # Initialize trainer  
    trainer = MNISTCFGTrainer(path = path, eta=0.35, null_label=10)  
  
    # losses, steps = trainer.train(model=dit, num_steps = 20000, lr=0.4e-3, batch_size=256, ckpt_every=1000)    
    losses, steps = trainer.train(model=dit, num_steps = 1200, lr=0.2e-3, batch_size=64, ckpt_every=1000)  
  
    plt.plot(steps, losses)  
    plt.xlabel("Step")  
    plt.ylabel("Loss")  
    plt.title("Loss vs. Step")  
    plt.show()  
  
    # Play with these!  
    samples_per_class = 10  
    num_timesteps = 100  
    guidance_scales = [1.0, 3.0, 5.0]  
  
    visualize_output(  
        model=dit,  
        path=path,  
        samples_per_class=samples_per_class,  
        num_timesteps=num_timesteps,  
        guidance_scales=guidance_scales,  
    )  
    plt.show()
```



## 5. VAE.py


```
图像 x              (B, 1, 32, 32)
  ↓ Encoder
潜变量均值 μ_z       (B, 128, 4, 4) + 标量 ℓ_z
  ↓ 重参数化采样
潜变量 z            (B, 128, 4, 4)
  ↓ Decoder
重建均值 μ_x         (B, 1, 32, 32) + 标量 ℓ_x
  ↓
高斯重建损失 + β × KL 散度
```


### 5.1 潜空间生成范式---变分自编码器 (VAE) 
#### 5.1.1 维数灾难与两阶段分离

直接在像素空间训练高分辨率流模型（如 $1024 \times 1024 \times 3 \approx 3.1 \times 10^6$ 维）在显存与吞吐上不可行。
- **第一阶段（感知压缩）**：训练 VAE / GAN 自编码器，将空间冗余压缩至低维潜空间（例如下采样 8 或 16 倍）。
- **第二阶段（语义生成）**：在紧凑且几何结构规整的潜变量空间中训练 Flow/Diffusion 模型。

#### 5.1.2 VAE 数学目标与重参数化

设编码器为 $q_\phi(z \mid x) = \mathcal{N}(\mu_\phi(x), \operatorname{diag}(\sigma_\phi^2(x)))$，解码器为 $p_\theta(x \mid z)$，先验为 $p_{\text{prior}}(z) = \mathcal{N}(0, I_k)$。
- **证据下界损失 (ELBO)**：

  $$\mathcal{L}_{\text{VAE}}(\phi, \theta) = \underbrace{\mathbb{E}_{x, z \sim q_\phi(z \mid x)} \left[ \frac{1}{2\sigma^2} \| x - \mu_\theta(z) \|^2 \right]}_{\text{重建误差 (Distortion)}} + \beta \underbrace{\mathbb{E}_x \left[ D_{\text{KL}}\big( q_\phi(z \mid x) \,\|\, \mathcal{N}(0, I) \big) \right]}_{\text{潜空间正则项 (Rate)}}$$

- **重参数化技巧 (Reparameterization Trick)**：

  $$z = \mu_\phi(x) + \sigma_\phi(x) \odot \epsilon, \quad \epsilon \sim \mathcal{N}(0, I_k)$$

  将随机性从计算图中剥离，使得梯度能够反向传播至编码器参数 $\phi$。

#### 5.1.3 Rate-Distortion Pareto Frontier

- **过小 $\beta$（高 Rate，极低 Distortion）**：重建保真度高，但潜空间分布形态极不规则，导致下游扩散模型难以拟合。
- **过大 $\beta$（极低 Rate，高 Distortion）**：潜空间趋于完美高斯，但解码出现严重后验崩溃，图像模糊。
- **黄金点**：位于 Pareto 边界的弯折拐角点（"Knee" 区域），利用轻微的潜空间正则约束确保生成模型易学，同时借助感知损失（Perceptual Loss）保留图像高频细节。



### 5.2 `ResidualBlock`

> 在原特征上学习局部修正

**`ResidualBlock.__init__(channels, act=nn.SiLU)`**

构建一条保持形状的残差分支。`GroupNorm(1, channels)` 对每张图的全部 $C\times H\times W$特征计算均值和方差，归一化后再应用逐通道的仿射参数。统计量来自单个样本，因此不依赖批次中其他图像。`3×3` 卷积读取邻域信息，`1×1` 卷积在每个位置混合通道。两层卷积的输入和输出通道数都是 `channels`。

**`ResidualBlock.forward(x)`**

输入和输出均为 $(B,C,H,W)$。先保留 `x_skip`，再计算：

$$R(x)=x+ \operatorname{Conv}_{1\times1} \left( \operatorname{SiLU}\left( \operatorname{Conv}_{3\times3}(\operatorname{GN}(x)) \right) \right), \qquad \operatorname{SiLU}(a)=a\,\sigma(a). $$

残差学习把模块写成 $x+F(x)$，让卷积分支学习对原特征的修正。这里第二层卷积的权重和偏置都初始化为零，所以 **模块刚创建时 $R(x)=x$**。沿着实际 `forward` 读，这一块只执行一次归一化和一次激活，分别位于第一层卷积之前和两层卷积之间。

```python
class ResidualBlock(nn.Module):  
  """ Two applications of LN + convolution + non-linearity + residual connection """  
  def __init__(self, channels: int, act: nn.Module = nn.SiLU):  
    super().__init__()  
  
    # Init norm, convolutions, and activations  
    # raise NotImplementedError("Fill me in!")    
    self.norm = nn.GroupNorm(num_groups=1, num_channels=channels)  
    self.conv1 = nn.Conv2d(channels, channels, kernel_size=3, padding=1)  
    self.conv2 = nn.Conv2d(channels, channels, kernel_size=1)  
    self.act = act()  
  
    # Initialize the second convolution to zero - stabilizes training early on!  
    nn.init.zeros_(self.conv2.weight)  
    nn.init.zeros_(self.conv2.bias)  
  
  def forward(self, x: torch.Tensor):  
    # Res init  
    x_skip = x  
  
    # Norm  
    # raise NotImplementedError("Fill me in!")    
    x = self.norm(x)  
  
    # First convolution  
    # raise NotImplementedError("Fill me in!")    
    x = self.conv1(x)  
  
    # Activation  
    # raise NotImplementedError("Fill me in!")    
    x = self.act(x)  
  
    # Second convolution  
    # raise NotImplementedError("Fill me in!")   
    x = self.conv2(x)  
  
    # Return residual connection  
    return x_skip + x
```


### 5.3 `AttnBlock`

> 让远处的图像位置交换信息

**`AttnBlock.__init__(channels)`**

将图像每个空间位置视为一个 token，token 的特征维度为 $C=\texttt{channels}$。模块包含两个 `LayerNorm(C)`、一个四头自注意力 `MHA`，以及逐 token 运行的前馈网络$C\to4C\to C$。这里的 `LayerNorm` 对**每个位置的通道向量**计算统计量，上一节的 `GroupNorm(1, C)` 则对**整张特征图**计算统计量。

**`AttnBlock.forward(x)`**

先把$(B,C,H,W)$ 重排为 $(B,HW,C)$，再执行两个归一化在前的残差子层：

$$ T'=T+\operatorname{MHA}(\operatorname{LN}_1(T)), \qquad T''=T'+\operatorname{MLP}(\operatorname{LN}_2(T')). $$

`MHA` 来自 `DiT.py`。每个头从同一序列投影得到 \(Q,K,V\)，计算：

$$A=\operatorname{softmax}\!\left(\frac{QK^\top}{\sqrt{C/4}}\right), \qquad O=AV. $$

对于$32\times32$特征图，每个头的注意力矩阵是$1024\times1024$。矩阵的一行描述一个位置如何汇总各位置的特征，因此数字上相隔较远的两段笔画也可以直接交换信息。相比之下，前馈网络只在**每个 token 内部**变换通道。最后，代码把序列恢复为 $(B,C,H,W)$。

```python
class AttnBlock(nn.Module):  
  def __init__(self, channels: int):  
    super().__init__()  
  
    # Reshape  
    self.reshape1 = Rearrange('b c h w -> b (h w) c')  
  
    # Norm + attention  
    self.norm1 = nn.LayerNorm(channels)  
    self.attn = MHA(channels, heads=4)  
  
    # Norm + ff     
    self.norm2 = nn.LayerNorm(channels)  
    self.ff = MLP([channels, 4 * channels, channels])  
  
  def forward(self, x: torch.Tensor):  
    b, c, h, w = x.shape  
    x = self.reshape1(x)  
  
    # Attention + residual connection     
    x_skip = x  
    x = x_skip + self.attn(self.norm1(x))  
  
    # Feedforward + residual connection  
    x_skip = x  
    x = x_skip + self.ff(self.norm2(x))  
  
    return rearrange(x, 'b (h w) c -> b c h w', h=h, w=w)
```

### 5.4 `EncoderBlock`

> 提取特征并可选地下采样

**`EncoderBlock.__init__(in_channels, downsample_channels=None)`**

构建两个 `ResidualBlock` 和一个 `AttnBlock`，三者都保持形状。如果指定了 `downsample_channels`，再加入一层核大小为 3、步长为 2、填充为 1 的卷积，同时降低空间分辨率并改变通道数。

**`EncoderBlock.forward(x)`**

按 `res1 → res2 → attn → 可选下采样` 的顺序处理。对当前使用的偶数边长，下采样卷积将 $H\times W$ 变为 $H/2\times W/2$。例如，`EncoderBlock(16, 32)` 的形状变化是：

$$ (B,16,32,32) \xrightarrow{\text{残差块与注意力}} (B,16,32,32) \xrightarrow{\text{步长为 2 的卷积}} (B,32,16,16). $$

当 `downsample_channels=None` 时，输出保持输入尺寸。编码器最后一块采用这个配置。

```python
class EncoderBlock(nn.Module):  
  def __init__(self, in_channels: int, downsample_channels: Optional[int] = None):  
    super().__init__()  
    # raise NotImplementedError("Fill me in!")  
    self.res1 = ResidualBlock(in_channels)  
    self.res2 = ResidualBlock(in_channels)  
    self.attn = AttnBlock(in_channels)  
  
    if downsample_channels is not None:  
      self.downsample = nn.Conv2d(  
          in_channels=in_channels,  
          out_channels=downsample_channels,  
          kernel_size=3,  
          stride=2,  
          padding=1,  
      )  
    else:  
      self.downsample = None  
  
  def forward(self, x: torch.Tensor):  
    # raise NotImplementedError("Fill me in!")  
    x = self.res1(x)  
    x = self.res2(x)  
    x = self.attn(x)  
  
    if self.downsample is not None:  
      x = self.downsample(x)  
  
    return x
```

### 5.5 `Encoder`

> 从图像得到潜变量分布的参数

**`Encoder.__init__(in_channels, hidden_channels)`**

入口使用 `3×3` 卷积把图像通道数从 1 提升到 16。`hidden_channels=[16,32,64,128]` 时，`zip` 产生四组配置 `(16,32)`、`(32,64)`、`(64,128)` 和 `(128,None)`。它们存入 `ModuleList`，使各层权重注册到模型中。均值输出层使用 `GroupNorm` 和 `1×1` 卷积，另外定义一个从 0 开始训练的**标量** `logvar`。

**`Encoder.forward(x)`**

执行入口卷积和四个编码块，再输出 `z_mean, self.logvar`。主要形状为：

$$ 1\times32\times32 \to16\times32\times32 \to32\times16\times16 \to64\times8\times8 \to128\times4\times4. $$

将潜变量展平后，这两个输出定义了近似后验：

$$ q_\phi(z\mid x)= \mathcal N\!\left(\mu_z(x),e^{\ell_z}I_{2048}\right), \qquad \ell_z=\texttt{z\_logvar}. $$

可以把返回值理解为：**每张图有自己的潜变量均值，所有图像和潜变量坐标共享一个方差。** `forward` 到这里已经定义了分布，具体采样由 `VAE.forward()` 完成。空间尺寸由 $32\times32$缩至 $4\times4$，同时通道增至 128，因此潜变量的总元素数为 2048，输入像素数为 1024。

```python
class Encoder(nn.Module):  
  def __init__(self, in_channels: int, hidden_channels: list[int]):  
    super().__init__()  
  
    # Initial conv2d  
    self.init_conv = nn.Conv2d(in_channels = in_channels, out_channels = hidden_channels[0], kernel_size=3, padding=1, stride=1)  
  
    # Initialize channels  
    ch_in = hidden_channels  
    ch_out = hidden_channels[1:] + [None]  
    blocks = []  
    for in_c, out_c in zip(ch_in, ch_out):  
      blocks.append(EncoderBlock(in_c, out_c))  
    self.blocks = nn.ModuleList(blocks)  
  
    # Predict z_mean 均值  
    z_dim = hidden_channels[-1]  
    self.z_mean = nn.Sequential(  
      nn.GroupNorm(1, z_dim),  
      nn.Conv2d(in_channels = z_dim, out_channels = z_dim, kernel_size=1, stride=1, padding=0),  
    )  
  
    # Scalar log-variance 方差  
    self.logvar = nn.Parameter(torch.zeros(()))  
  
  def forward(self, x: torch.Tensor):  
    # raise NotImplementedError("Fill me in!")  
    x = self.init_conv(x)  
  
    for block in self.blocks:  
        x = block(x)  
  
    z_mean = self.z_mean(x)  
    return z_mean, self.logvar
```


### 5.6 `DecoderBlock`

> 处理低分辨率特征并可选地放大

**`DecoderBlock.__init__(in_channels, upsample_channels=None)`**

与编码块一样，先构建两个残差块和一个注意力块。如果指定 `upsample_channels`，则建立 `最近邻上采样 → 3×3 卷积`。最近邻上采样把每个空间位置复制到 $2\times2$ 区域，随后卷积在放大后的网格上混合邻域特征并转换通道。

**`DecoderBlock.forward(x)`**

顺序为 `res1 → res2 → attn → 可选上采样`。例如，`DecoderBlock(128, 64)` 先在 $(B,128,4,4)$上计算残差和注意力，随后输出 $(B,64,8,8)$。最后一块的 `upsample_channels=None`，因此保留$32\times32$的空间尺寸。

```python
class DecoderBlock(nn.Module):  
  def __init__(self, in_channels: int, upsample_channels: Optional[int] = None):  
    super().__init__()  
    self.res1 = ResidualBlock(in_channels)  
    self.res2 = ResidualBlock(in_channels)  
    self.attn = AttnBlock(in_channels)  
    if upsample_channels is not None:  
      self.upsample = nn.Sequential(  
        nn.Upsample(scale_factor=2, mode='nearest'),  
        nn.Conv2d(in_channels=in_channels, out_channels=upsample_channels, kernel_size=3, padding=1, stride=1),  
      )  
    else:  
      self.upsample = None  
  
  def forward(self, x: torch.Tensor):  
    # raise NotImplementedError("Fill me in!")  
    x = self.res1(x)  
    x = self.res2(x)  
    x = self.attn(x)  
  
    if self.upsample is not None:  
        x = self.upsample(x)  
  
    return x
```
`

### 5.7 `Decoder`

> 从潜变量得到重建图像的分布

**`Decoder.__init__(out_channels, hidden_channels)`**

接收逆序的通道列表 `[128,64,32,16]`，依次构建 `(128,64)`、`(64,32)`、`(32,16)` 和 `(16,None)` 四个解码块。输出头用 `GroupNorm` 和 `1×1` 卷积将 16 通道特征变为单通道图像均值。解码器有一个独立于编码器的可学习标量 `logvar`。

**`Decoder.forward(x)`**

形参 `x` 在这里接收的是潜变量 \(z\)。主要形状变化为：

$$ 128\times4\times4 \to64\times8\times8 \to32\times16\times16 \to16\times32\times32 \to1\times32\times32. $$

返回 `x_mean, self.logvar`，对应解码似然：

$$ p_\psi(x\mid z) = \mathcal N\!\left(\mu_x(z),e^{\ell_x}I_{1024}\right). $$

`x_mean` 是**标准化图像空间**中的重建均值，输出头为线性卷积。训练重建图和插值图时，代码直接取这个均值作为可视化内容。

```python
class Decoder(nn.Module):  
  def __init__(self, out_channels: int, hidden_channels: list[int]):  
    super().__init__()  
  
    # Initialize channels  
    ch_in = hidden_channels  
    ch_out = hidden_channels[1:] + [None]  
    blocks = []  
    for in_c, out_c in zip(ch_in, ch_out):  
      blocks.append(DecoderBlock(in_c, out_c))  
    self.blocks = nn.ModuleList(blocks)  
  
    # Predict mean  
    x_dim = hidden_channels[-1]  
    self.x_mean = nn.Sequential(  
      nn.GroupNorm(1, x_dim),  
      nn.Conv2d(in_channels = x_dim, out_channels = out_channels, kernel_size=1, stride=1, padding=0),  
    )  
  
    # Scalar log-variance  
    self.logvar = nn.Parameter(torch.zeros(()))  
  
  def forward(self, x: torch.Tensor):  
    # raise NotImplementedError("Fill me in!")  
    for block in self.blocks:  
        x = block(x)  
  
    x_mean = self.x_mean(x)  
    return x_mean, self.logvar
```

### 5.8 `VAE`

> 组织编码、采样和解码

**`VAE.__init__(data_channels, hidden_channels, beta=0.1)`**：创建编码器和解码器，并保存 KL 权重 `beta`。构造函数默认值为 `0.1`，本文件的训练入口显式传入 `10.0`。报告实验配置时应记录实际传入值。

**`VAE.encode(x)`**：调用 `_encoder(x)`，返回 $(\mu_z,\ell_z)$。例如，下游潜空间生成器可先取出这两个参数，再自行决定使用后验样本还是后验均值。

**`VAE.decode(z)`**：调用 `_decoder(z)`，返回 $(\mu_x,\ell_x)$。潜空间生成流程得到新的 $z$ 后可以直接调用这个接口，取 $\mu_x$ 生成图像。

**`VAE.forward(x)`**：完成一次 `encode → sample → decode`。采样使用重参数化：

$$ z=\mu_z(x)+e^{\ell_z/2}\epsilon, \qquad \epsilon\sim\mathcal N(0,I). $$

`logvar` 表示 $\log\sigma^2$，所以 `exp(0.5 * z_logvar)` 得到的是标准差。举例来说，若某个坐标的 $\mu_z=1$、$\ell_z=\log 4$，本次抽到的 $\epsilon=-0.5$，则 $z=1+2(-0.5)=0$。随机数 $\epsilon$与模型参数无关，采样结果对 $\mu_z,\ell_z$仍可求导，因此重建损失可以通过 $z$ 传回编码器。函数最终返回四个分布参数 `z_mean, z_logvar, x_mean, x_logvar`。

**`VAE.compute_loss(z_mean, z_logvar, x_mean, x_logvar, x_true)`**：VAE 的生成模型假设 $z\sim\mathcal N(0,I)$，然后由 $p_\psi(x\mid z)$ 生成图像。由于边缘似然 $p_\psi(x)=\int p_\psi(x\mid z)p(z)\,\mathrm dz$通常难以直接计算，VAE 使用编码器提供的 $q_\phi(z\mid x)$，优化证据下界（ELBO）：

$$\operatorname{ELBO}(x)= \mathbb E_{q_\phi(z\mid x)}[\log p_\psi(x\mid z)] -D_{\mathrm{KL}}\bigl(q_\phi(z\mid x)\|p(z)\bigr). $$

代码用每张图的一次潜变量采样估计重建期望，并给 KL 项乘以 $\beta$。当$\beta=1$ 时对应标准负 ELBO 的估计，当本实验取 $\beta=10$时，对应加权目标。
1. **KL 项衡量编码分布与标准高斯先验的差距。** 对第 $i$ 张图：

$$K_i= \frac12\sum_{j=1}^{2048} \left[ \mu_{z,ij}^2+e^{\ell_z}-1-\ell_z \right]. $$

    `z_logvar` 是标量，因此方差相关项广播到每个潜变量坐标。若 $\mu_z=0,\ell_z=0$，编码分布恰好等于先验，$K_i=0$。
2. **重建项是高斯负对数似然。** 对第 $i$ 张图：

    $$ R_i= \frac12\sum_{k=1}^{1024} \left[ \log(2\pi)+\ell_x+ (x_{ik}-\mu_{x,ik})^2 e^{-\ell_x} \right]. $$

    平方误差由逆方差 $e^{-\ell_x}$加权，同时损失含有 $\ell_x$项。因为 $\ell_x$ 也参与学习，报告损失时可以把重建负对数似然和像素 MSE 分开记录。
最终代码对每张图的潜变量维度或像素维度求和，再对批次求平均：

$$ \mathcal L=\frac1B\sum_{i=1}^{B}(R_i+\beta K_i) $$

`flatten(start_dim=1).sum(dim=1)` 对应单张图求和，`.mean()` 对应批次平均。这个归约方式决定了损失数值的尺度。


```python
class VAE(nn.Module):  
  def __init__(self, data_channels: int, hidden_channels: list[int], beta: float = 0.1):  
    super().__init__()  
    self.beta = beta  
  
    # Encoder  
    self._encoder = Encoder(data_channels, hidden_channels)  
  
    # Decoder  
    self._decoder = Decoder(data_channels, list(reversed(hidden_channels)))  
  
  def encode(self, x: torch.Tensor):  
    return self._encoder(x)  
  
  def decode(self, z: torch.Tensor):  
    return self._decoder(z)  
  
  def forward(self, x: torch.Tensor):  
    z_mean, z_logvar = self.encode(x)  
    z = z_mean + torch.exp(0.5 * z_logvar) * torch.randn_like(z_mean)  
    x_mean, x_logvar = self.decode(z)  
    return z_mean, z_logvar, x_mean, x_logvar  
  
  def compute_loss(self, z_mean: torch.Tensor, z_logvar: torch.Tensor, x_mean: torch.Tensor, x_logvar: torch.Tensor, x_true: torch.Tensor):  
    """ See display 85 from the text """  
    # KL loss    # raise NotImplementedError("Fill me in!")  
    # Reconstruction loss    # raise NotImplementedError("Fill me in!")  
    # KL loss: KL(q(z|x) || N(0, I))    
    kl_per_sample = 0.5 * (  
    z_mean.square() + torch.exp(z_logvar) - 1 - z_logvar  
        ).flatten(start_dim=1).sum(dim=1)  
    kl_loss = self.beta * kl_per_sample.mean()  
  
    # Reconstruction loss: Gaussian negative log-likelihood  
    recon_per_element = 0.5 * (  
    math.log(2 * math.pi)  
    + x_logvar  
    + (x_true - x_mean).square() * torch.exp(-x_logvar)  
)  
    recon_per_sample = recon_per_element.flatten(start_dim=1).sum(dim=1)  
    recon_loss = recon_per_sample.mean()  
  
  
    return kl_loss + recon_loss

```

### 5.19 Training VAE


- VAE num_steps=1000
隐空间插值结果展示了从数字 0 到数字 1 的平滑过渡。随着隐向量在两个样本的 latent 编码之间线性插值，图像形态连续渐变，没有出现突兀的跳变，说明$\beta$-VAE 学习得到了连续的隐空间。由于本实验取 $\beta$=10，KL 散度的约束较强，隐空间分布更贴近标准正态分布，保障了插值的流畅性，但也造成重建图像清晰度下降。

![[Pasted image 20261113185351.png]]

- VAE num_steps=1000

![[step_001000_output.png]]

- VAE num_steps=5000

![[Pasted image 20261113185307.png]]

## 6. Latent_CFG_VAE_DiT.py

### 6.1 从像素生成到学习表示上的生成

高分辨率图像包含大量局部细节，逐像素建模成本高。Latent Diffusion 的代表思路是先学习图像表示，再在该表示上运行生成过程，最终解码回图像。

> 本项目借鉴其组合方式，但第一阶段是带全局方差的教学 VAE，没有实现原始 LDM 全部感知损失、对抗训练或复杂条件机制。应写为“VAE 潜空间上的 Flow Matching”，而非完整复现某大型潜扩散系统。

### 6.2 潜空间 DiT

VAE 的 KL 正则不保证实际编码分布恰好为标准高斯。类别信息、重建需求和有限模型容量都会使聚合后验具有复杂结构：

$$q_\phi(z\mid y)=\int q_\phi(z\mid x)p_{\mathrm{data}}(x\mid y)\,\mathrm dx. $$

潜空间 DiT 要学习的正是这一类别条件分布，而不是简单重复从 $\mathcal N(0,I)$ 抽样。VAE 负责表示与解码，DiT 负责组织该表示中的生成分布。

### 6.3 两阶段训练与冻结

第一阶段优化 VAE。第二阶段固定编码器与解码器，然后：

$$x,y\sim p_{\mathrm{data}},\quad z\sim q_\phi(z\mid x),\quad\epsilon\sim\mathcal N(0,I),$$

$$Z_t=tz+(1-t)\epsilon,$$

$$\mathcal L_{\mathrm{latent}}=\mathbb E\|u_\theta(Z_t,t,\widetilde y)-(z-\epsilon)\|^2. $$

代码在 `torch.no_grad()` 中编码，并冻结 VAE 参数。这不仅节省梯度记录，也保持第二阶段的目标表示不随训练改变。`eval()` 与 `requires_grad_(False)` 并非同义：前者控制训练/评估行为，后者控制参数是否求梯度。

### 6.4 “潜空间压缩”的含义

图像元素数为 $1\times32\times32=1024$，潜变量元素数为 $128\times4\times4=2048$。因此当前实现降低空间分辨率，却增加了总标量数量。

DiT 的 token 数从 64 降为 16，在隐藏维度相同条件下，注意力中的 $N^2$ 项变成原来的 $1/16$，逐 token 前馈项约变为 $1/4$。但输入投影、输出通道及 VAE 编解码仍有成本，不能把注意力矩阵缩小直接写成整个系统加速 16 倍。

### 6.5 解码均值与完整概率模型的差别

理论组合可以写成：

$$p_{\theta,\psi}(x\mid y)=\int p_\psi(x\mid z)p_\theta(z\mid y)\,\mathrm dz.$$

当前采样在生成潜变量后仅使用 $\mu_\psi(z)$，不从 $p_\psi(x\mid z)$ 添加输出噪声。因此交付图像来自解码均值的映射分布，与完整高斯似然采样有所不同。这是代码的明确选择，不应省略。


| 模型 | 输入 | patch | token 数 | token 维度 |
|---|---|---|---|---|
| 像素 DiT | $1\times32\times32$ | $4\times4$ | 64 | 256 |
| 潜空间 DiT | $128\times4\times4$ | $1\times1$ | 16 | 256 |

潜空间一个 $1\times1$ patch 仍含 128 个通道，所以不是一个原始灰度像素。

![[CFG_VAE_DiT_loss_5000.png]]

- steps_num=1000

![[step_ 1000_output.png]]

- steps_num=3000

![[step_ 3000_output.png]]

- steps_num=5000

![[step_ 5000_output 1.png]]


## 7. Adapter.py

> 本实验的Adapter仅为简单的流程架构，并未针对特定问题进行设计。

### 7.1 PEFT 的背景

全参数微调需要为所有权重保存梯度和优化器状态，多任务还可能需要保存多套模型。参数高效微调 PEFT（Parameter-Efficient Fine-Tuning）限制更新参数，使任务适配更节省存储与优化资源。

Houlsby 等人的NLP Adapter工作展示了冻结预训练网络、插入小模块进行任务适配的路线[Adapter](https://arxiv.org/abs/1902.00751)。后来 LoRA 通过低秩矩阵表示权重更新，形成另一种参数高效方法[LoRA](https://arxiv.org/abs/2106.09685)。

本项目采用的是非线性瓶颈残差 Adapter，不是 LoRA，也不是用于边缘图控制的 T2I-Adapter。
### 7.2 当前 Adapter 的结构

对一个 token $h\in\mathbb R^d$：

$$A_\varphi(h)=h+W_u\operatorname{SiLU}(W_dh+b_d)+b_u. $$

其中 $W_d\in\mathbb R^{r\times d},W_u\in\mathbb R^{d\times r}$。瓶颈宽度 $r$ 限制中间容量；残差让模型可在原表示附近学习修正。

代码通过 forward hook，将模块挂在最后两个完整 Transformer 层的输出之后。

![[Pasted image 20261114110024.png]]

### 7.3 参数量与初始化

单个模块参数量为：

$$P=dr+r+rd+d=2dr+r+d.$$

取 $d=256,r=16$：

$$P=8464,\qquad P_{\mathrm{two}}=16928.$$

由于 $W_u,b_u$ 零初始化，初始时 $A(h)=h$。因此添加模块之初，模型函数与原主干一致。

设残差为 $a=W_u\sigma(W_dh+b_d)$，则下投影梯度包含 $W_u^\top$ 因子。初始化瞬间，下投影的损失梯度为零，而上投影可以更新；随后下投影开始收到有效梯度。这解释了零初始化的训练启动方式，不能据首步梯度为零认定 Adapter 失效。

### 7.4 冻结主干仍然需要传播梯度

优化目标为：

$$\min_\varphi\mathbb E\|u_{\theta_0,\varphi}(Z_t,t,\widetilde y)-u_t^{\mathrm{ref}}\|^2,$$

其中 $\theta_0$ 固定。冻结权重意味着不更新该权重，不代表把该运算从计算图删除。为了更新较早的 Adapter，仍需通过后续冻结层计算输入梯度。

当前优化器筛选 `requires_grad=True` 的参数，属于真正的参数子集优化。但检查点保存完整 `state_dict()`，其中仍包含主干；因此“训练参数少”和“当前保存文件小”不能画等号。

```python
class ResidualAdapter(nn.Module):  
    def __init__(self, dim, width):  
        super().__init__()  
        self.down = nn.Linear(dim, width)  
        self.activation = nn.SiLU()  
        self.up = nn.Linear(width, dim)  
        nn.init.zeros_(self.up.weight)  
        nn.init.zeros_(self.up.bias)  
  
    def forward(self, tokens):  
        return tokens + self.up(self.activation(self.down(tokens)))  
  
def add_adapters(model, width):  
    for layer in model.dit.layers[-2:]:  
        layer.adapter = ResidualAdapter(layer.norm2.normalized_shape[0], width)  
  
  
def adapter_forward(layer, inputs, output):  
    return layer.adapter(output)  
  
  
def attach_adapters(model, width):  
    # 在最后两层的输出接入 Adapter，原 DiT 的 forward 无需改动。  
    layers = model.dit.layers[-2:]  
  
    if any(hasattr(layer, "adapter") for layer in layers):  
        raise ValueError("DiT 的最后两层已挂载 Adapter。")  
  
    add_adapters(model, width)  
    for layer in layers:  
        layer.register_forward_hook(adapter_forward)
```

### 7.5 AdapterLatentCFGTrainer

```python
vae = VAE(  
    data_channels=1,  
    hidden_channels=[16, 32, 64, 128],  
    beta=10.0,  
).to(device)  
  
# 加载权重 训练1000步----Problem  
vae_ckpt_path = "runs/delicate-orange-f3910e8f/step_005000_model.pt"  
vae.load_state_dict(torch.load(vae_ckpt_path, map_location=device))  
# 冻结 VAEvae.eval()  
vae.requires_grad_(False)  
  
  
#  
dit = DiffusionTransformerFlowModel(  
    img_size=img_size,  
    patch_size=1,  
    num_layers=8,  
    c=c,  
    dim=256,  
    heads=8,  
    final_dim=10,  
    n_classes=11,  
)  
dit_ckpt_path = "runs/white-jackfruit-8bda7561/step_  5000_model.pt"  
dit.load_state_dict(torch.load(dit_ckpt_path, map_location=device, weights_only=True))  
dit.requires_grad_(False)  
attach_adapters(dit, width=16)  
dit.to(device)  
print(f"Trainable Adapter parameters: {sum(p.numel() for p in dit.parameters() if p.requires_grad):,}")  
  
#  
path = GaussianConditionalProbabilityPath(  
    p_data=None,  
    p_simple_shape=[c, img_size, img_size],  
    alpha=LinearAlpha(),  
    beta=LinearBeta(),  
).to(device)  
  
mnist = MNISTSampler().to(device)  
  
trainer = AdapterLatentCFGTrainer(  
    mnist=mnist,  
    vae=vae,  
    path=path,  
    eta=0.35,  
    null_label=10,  
)  
  
losses, steps = trainer.train(model=dit, num_steps=5005, lr=0.4e-3, batch_size=64, ckpt_every=500)
```

![[Adapter_Loss_5000.png]]

- steps_num=1000

![[step_ 1000_adapter_output.png]]

- steps_num=3000

![[step_ 3000_adapter_output.png]]

- steps_num=5000

![[step_ 5000_adapter_output.png]]


## 8. 数值采样、优化

### 8.1 Euler 方法

从导数定义出发，短时间内用当前位置速度近似变化：

$$X_{k+1}=X_k+\Delta t_k\,u_{\mathrm{CFG}}(X_k,t_k,y). $$

这就是显式 Euler。对足够平滑且稳定的问题，局部截断误差为 $O(\Delta t^2)$、累计全局误差为 $O(\Delta t)$。它实现简单，但较少步数和较强 CFG 可能增加误差。

当前潜空间比较使用 100 个时间点覆盖 $[0,0.999]$，因此为 99 次更新。代码每步分别运行条件与无条件预测，即 198 次 DiT 前向调用，另加解码。即使 $w=1$，当前实现仍计算两个分支；数学上可以省略其中一个，但代码尚未做这种优化。

NFE（Number of Function Evaluations）指函数评估次数。比较采样效率时应明确统计速度场调用还是网络分支调用，不能把“100 个时间点”直接写成“100 次网络计算”。

### 8.2 AdamW 

Adam 类优化器使用梯度一阶和二阶滑动统计：

$$m_s=b_1m_{s-1}+(1-b_1)g_s,\qquad v_s=b_2v_{s-1}+(1-b_2)g_s^2.$$

偏差修正后，AdamW 将权重衰减与自适应梯度更新分开：

$$\theta_{s+1}=(1-\lambda_s\rho)\theta_s-\lambda_s\frac{\widehat m_s}{\sqrt{\widehat v_s}+\delta}.$$

生成模型通用训练器设置权重衰减 $\rho=10^{-4}$。默认学习率线性预热 500 步：

$$\lambda_s=\lambda_{\max}\min((s+1)/500,1). $$

预热的用途是减小训练初期的大更新。当前潜空间 DiT 和 Adapter 各训练 600 步，因此 500 步预热占比很高。应记录实际学习率轨迹，而非只报告最大值。


## 9.实验记录

### 9.1各个模块的具体运行记录

![[b76cc22ee37f0f0387ed83c9f32edf40.jpg]]

### 9.2 Adapter的对比

- 仅为分类器判断实验
![[Pasted image 20261111185405.png]]