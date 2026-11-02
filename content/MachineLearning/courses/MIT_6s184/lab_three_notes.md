# Lab 3：图像条件生成实验笔记

> 对应实验：[lab_three.ipynb](./lab_three.ipynb)  
> 主题：MNIST 条件生成、Classifier-Free Guidance（CFG）、Diffusion Transformer（DiT）、VAE 与潜空间生成。

## 1. 实验目标与整体流程

Lab 3 将前面实验中的二维分布生成扩展到 MNIST 图像。图像是高维张量，实验以 `32 × 32` 灰度图像为例，并逐步构建两种生成方案：

1. **像素空间生成**：直接在 `1 × 32 × 32` 图像空间训练带类别条件的向量场。
2. **潜空间生成**：先训练 VAE，将图像压缩到小尺寸潜变量，再在潜空间训练条件生成模型，最后由 VAE 解码回图像。

两种方案都使用条件 Flow Matching，并用 CFG 在采样时增强类别条件。

```text
MNIST 图像 x、类别 y
        │
        ├── 像素空间：Gaussian probability path → DiT → 生成图像
        │
        └── VAE 编码器 → 潜变量 z → latent DiT → VAE 解码器 → 生成图像
```

## 2. 基础组件与张量约定

### 2.1 Sampleable 与 MNISTSampler

- `Sampleable.sample(n)` 从无标签分布采样数据。
- `LabeledSampleable.sample(n)` 返回 `(samples, labels)`。
- `MNISTSampler` 返回图像 `x`（形状 `(b, 1, 32, 32)`）和类别 `y`（形状 `(b,)`）。
- 图像经过缩放并标准化；因此训练张量的数值范围不一定是 `[0, 1]`。展示图片时通常还需归一化或反标准化。

### 2.2 Gaussian 条件概率路径

实验中的高斯路径可写成：

$$
x_t = \alpha_t z + \beta_t \epsilon,\qquad \epsilon \sim \mathcal{N}(0,I),
$$

其中 `z` 是数据样本或 VAE 潜变量，`t` 是时间，$\epsilon$是简单分布中的噪声。模型学习向量场 $u_t^\theta(x_t \mid y)$，目标是路径对应的条件向量场 $u_t^{\mathrm{ref}}(x_t \mid z)$。

采用 $\alpha_t=t$、$\beta_t=1-t$ 时：

$$
x_t = tz + (1-t)\epsilon,\qquad
u_t^{\mathrm{ref}}(x_t\mid z)=\frac{z-x_t}{1-t}.
$$

实现中通常将 $t$限制在 `[0, 1-eps)`，避免分母在 $t=1$处为零或过小。


## 3. Classifier-Free Guidance（CFG）

### 3.1 条件训练目标

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

### 3.2 推理时的引导

同一个模型分别使用真实标签和 null label 预测向量场：

$$
\tilde{u}_t^\theta(x\mid y)
=(1-w)u_t^\theta(x\mid\varnothing)+w u_t^\theta(x\mid y).
$$

- `w=1`：普通有条件预测，没有额外 CFG 放大。
- `w>1`：更强地强调类别条件，通常会提高类别一致性，但过大可能降低多样性或造成失真。
- $\eta$控制训练时 null label 样本的比例；`w` 控制生成时条件引导强度，二者作用阶段不同。

## 4. 像素空间 Diffusion Transformer

### 4.1 Fourier 时间编码

将标量时间 `t` 编码为正弦、余弦特征：

$$
t^{\mathrm{emb}} =
[\sin(2\pi w_i t),\,\cos(2\pi w_i t)]_{i=1}^{d/2}.
$$

这样网络能以多种频率感知时间位置，而不只是接收单一标量。实现时若 `t` 形状是 `(b, 1, 1, 1)`，传入编码器前需整理成 `(b,)`。

### 4.2 Patchifier 与 Depatchifier

- **Patchifier**：通过卷积（卷积核和步幅通常均为 `patch_size`）将图像划分为 patch，再展平为 token 序列 `(b, n, dim)`。
- token 数量为 $n=(H/p)(W/p)$。例如 `32 × 32` 图像、`patch_size=4` 时有 64 个 token。
- **Depatchifier**：将每个 token 映射到一个 patch 的像素特征，重排回二维特征图，再通过卷积输出图像通道。

### 4.3 多头自注意力（MHA）

输入形状为 `(b, n, dim)`。线性层生成 Q、K、V，将特征维拆成多个头，然后计算：

$$
\operatorname{Attention}(Q,K,V)
=\operatorname{softmax}\left(\frac{QK^\top}{\sqrt{d_h}}\right)V.
$$

最后将各头合并，并经过输出投影。必须满足 `dim % heads == 0`。

### 4.4 Diffusion Transformer Layer

每层包含注意力和前馈网络两个残差分支。条件向量由时间 embedding 与类别 embedding 相加得到，再通过 AdaLN 调制分支：

$$
\operatorname{modulate}(h,\gamma,\beta)
=h(1+\gamma)+\beta.
$$

每个分支还有门控值控制残差更新。AdaLN 最后一层零初始化时，初始门控和调制量为零，使层在训练初期接近恒等映射，有助于稳定训练。

完整模型的数据流：

```text
图像 x ─→ Patchifier ─→ 加位置编码 ─→ 多层 DiT ─→ Depatchifier ─→ 向量场输出
时间 t ─→ Fourier Encoder ─┐
类别 y ─→ Embedding ───────┴→ 条件向量 c
```

## 5. VAE：图像压缩与重建

### 5.1 结构

- **Encoder**：初始卷积后，通过多个编码器块；每个块包含残差块、注意力块，非末块还会下采样。最终输出潜变量均值 `z_mean` 和标量 `z_logvar`。
- **重参数化采样**：

$$
z=z_{\mathrm{mean}}+\exp(0.5\,z_{\mathrm{logvar}})\epsilon,\qquad
\epsilon\sim\mathcal{N}(0,I).
$$

- **Decoder**：通过多个解码器块逐步上采样，最终输出重建均值 `x_mean` 和标量 `x_logvar`。
- 残差块使用归一化、卷积、非线性激活和残差连接；第二个卷积零初始化时，残差块起始时接近恒等映射。
- 注意力块把空间位置视为 token，在像素位置之间建模关系。

常见设置 `hidden_channels=[16, 32, 64, 128]` 时，32×32 特征经三次下采样得到约 4×4 的潜空间特征，通道数为 128。

### 5.2 VAE 损失

损失由重建负对数似然和 KL 正则项组成：

$$
\mathcal{L}_{\mathrm{VAE}}
=\mathcal{L}_{\mathrm{recon}}+\beta\mathcal{L}_{\mathrm{KL}}.
$$

对角高斯后验 \(q(z\mid x)\) 与标准正态先验的 KL 项为：

$$
\mathcal{L}_{\mathrm{KL}}
=\frac12\sum_j
\left(\mu_j^2+\exp(\log\sigma^2)-1-\log\sigma^2\right).
$$

若解码器假设像素服从方差为 \(\exp(x_{\mathrm{logvar}})\) 的高斯分布，重建项为高斯负对数似然：

$$
\mathcal{L}_{\mathrm{recon}}
=\frac12\sum_i\left[
\log(2\pi)+x_{\mathrm{logvar}}
+(x_i-x_{\mathrm{mean},i})^2\exp(-x_{\mathrm{logvar}})
\right].
$$

通常对每张图像的所有元素求和，再对 batch 求均值。由于是对像素/潜变量维度求和，loss 数值可能达到数百或数千；不能脱离损失定义，仅凭数值大小判断错误。

## 6. 潜空间扩散模型

像素空间模型需要直接处理 32×32 图像；潜空间方案则先用训练好的 VAE 将其压缩，再对潜变量训练 DiT：

1. 固定已训练的 VAE。
2. 从 MNIST 采样图像和标签。
3. 在 `torch.no_grad()` 下编码图像，并从编码器给出的潜变量分布采样 `z`。
4. 在潜空间构造 Gaussian probability path，训练条件向量场和 null-label 分支。
5. 推理时从潜空间简单分布采样，使用 CFG ODE/数值积分生成潜变量，再由 VAE 解码为图像。

关键点：潜空间训练时 VAE 编码器不应参与梯度更新；推理时则要用相同的 VAE 解码生成的潜变量。

## 7. 训练曲线与实验现象

本次记录的潜空间实验使用了较小的运行设置：

```text
num_steps = 1000
lr = 1e-4
batch_size = 64
ckpt_every = 2000
潜变量形状约为 (128, 4, 4)
```

观察到 loss 从约 4110 缓慢下降到约 4040，同时有明显 batch 波动。这种整体下降是积极信号；波动可能来自每步使用不同随机 batch。每个样本的误差还会在 2048 个潜变量元素上求和，因此总损失约为几千并不意外。

由于 `ckpt_every=2000` 大于 `num_steps=1000`，这次运行不会在训练中触发 2000 步检查点，也就不会通过该回调保存中途样本图。loss 曲线本身不能证明生成图像已经正常；应同时检查 VAE 重建和采样结果。

## 8. `Fill me in!` 


### Part 2：CFG

- **CFG 训练损失**：采样 `(z, y)`，以概率 `eta` 将 `y` 换成 null label；采样 `t` 和 `x_t`，用模型预测向量场，并与 `path.conditional_vector_field(x_t, z, t)` 的目标计算平方误差。

- **MLP 条件向量场**：通过 `nn.Embedding` 把类别 `y` 映射成向量，与 `x`、时间 `t` 拼接后输入 MLP；输入维数是 `dim + class_dim + 1`。

CFG 训练损失的实现通常从数据批次 `(z, y)` 开始：以概率 `eta` 把标签替换为预留的 null label，再采样时间 `t` 和概率路径上的 `x_t`，用模型预测向量场，并对预测值与目标 `conditional_vector_field(x_t, z, t)` 计算均方误差。这样写是把连续时间上的 Flow Matching 目标转成可通过 minibatch 估计的训练损失。随机丢弃条件标签的关键作用，是让同一个网络同时学到有条件和无条件的向量场；推理时才能用二者组合实现 CFG。要特别区分：`eta` 是训练阶段丢弃标签的概率，`guidance_scale` 是采样时控制引导强度的参数，两者不是同一概念。

MLP 条件向量场把类别标签经过 `nn.Embedding` 映射成连续向量，再与当前状态 `x` 和时间 `t` 拼接后输入 MLP。它对应的学习重点是“条件如何进入网络”：类别不只是数据集元信息，而是预测函数的输入。检查实现时，重点核对标签 embedding 的 batch 维、`t` 是否能与 batch 对齐，以及首层输入维度是否为状态维、类别 embedding 维和时间维之和。通过这一步，后续 DiT 使用更复杂的条件调制时，就有了同一个基本认识。

### Part 3：Diffusion Transformer

- **FourierEncoder**：计算 `2*pi*weights*t`，拼接正弦和余弦特征，将标量时间变成高维表示。

- **Patchifier**：用 `kernel_size=stride=patch_size` 的卷积切分并投影图像；展平空间维，得到 `(batch, tokens, dim)`。

- **MHA**：线性映射得到 Q/K/V，拆分 heads，计算 `softmax(QKᵀ/sqrt(head_dim))V`，合并 heads 后做输出投影。

- **DiT layer**：条件向量生成两组 AdaLN 的 shift、scale 和 gate，分别调制注意力与前馈分支；两支都使用残差连接，调制层末层零初始化。

- **DiffusionTransformer**：建立可学习的位置编码和 `depth` 个 Transformer 层；先给 token 加位置编码，再逐层传递条件向量。

- **Depatchifier**：对 token 做归一化和 MLP 投影，将每个 token 映射成 patch 像素特征；重排回特征图，再卷积成图像通道。

- **完整 DiT flow model**：Fourier 编码时间、embedding 类别，两者相加作为条件；输入图像经过 patchifier、DiT、depatchifier，输出向量场。


FourierEncoder 将时间映射到一组不同频率的正弦、余弦特征（典型形式为 `sin(2*pi*weights*t)` 与 `cos(2*pi*weights*t)`）。单一标量时间虽然足以描述时刻，但 MLP 不一定容易直接从标量中学习快速变化的时间依赖；多频率编码提供了更丰富的基函数。这里要确认时间张量的形状和广播方式，并理解编码后的时间特征维度会影响后续条件层。

Patchifier / Depatchifier 负责在图像与 Transformer 序列之间转换。Patchifier 将图像切成不重叠 patch 并投影为 token，常见实现是卷积核大小和步长均为 `patch_size`，输出形状为 `(batch, n_tokens, dim)`；当图像高宽可被 patch 大小整除时，`n_tokens=(H/patch_size)*(W/patch_size)`。Depatchifier 则把每个 token 预测的 patch 特征重新排列回二维空间，再投影为图像形状的向量场。实现这两个模块的学习价值不只是张量变形：它让你看到 patch 大小如何影响 token 数、注意力计算量和可保留的空间细节。最常见的问题是 token 数、通道维或重排顺序不一致。

多头注意力（MHA）先由线性层生成 Q、K、V，再拆成多个 head，计算 `softmax(QK^T/sqrt(head_dim))V`，最后合并各 head 并投影。它让相距较远的图像 patch 也能交换信息，补足卷积偏重局部邻域的特点。核对时要确认 `dim` 能被 `heads` 整除，并检查拆分、转置和合并前后的维度；缩放项 `sqrt(head_dim)` 用来避免点积随维数变大而导致 softmax 过度饱和。

DiT layer 用条件向量产生 AdaLN 的 shift、scale 和 gate，调制注意力分支与前馈分支，并通过残差连接合并。时间和类别条件因此不只在输入端拼接一次，而能影响每个 Transformer block 的内部计算。位置编码则告诉 Transformer 每个 token 在图像中的位置；否则注意力本身并不知道序列 token 对应的空间布局。零初始化输出调制层或末层的设计，使模型初始时的残差更新较小，有利于稳定开始训练。完整的 DiffusionTransformerFlowModel 再把时间 Fourier 特征、类别 embedding、Patchifier、DiT blocks 和 Depatchifier 串起来，输入图像状态、时间和标签，输出同形状的向量场。到这里，填空训练的是系统集成能力：每个小模块局部正确还不够，接口形状、条件语义和最终输出也必须吻合 Flow Matching 目标。

### Part 4：VAE

- **ResidualBlock**：GroupNorm → 3×3 卷积 → 激活 → 1×1 卷积 → 输入残差；第二个卷积零初始化，使模块初始接近恒等映射。

- **AttnBlock**：把图像空间位置展平成 token，先做 LayerNorm + MHA + 残差，再做 LayerNorm + MLP + 残差，最后还原空间维。head 数必须整除 `channels`。

- **EncoderBlock / DecoderBlock**：核心都是两个残差块和一个注意力块；EncoderBlock 可选步幅 2 卷积下采样，DecoderBlock 可选上采样后卷积改变通道数。

- **Encoder / Decoder**：分别串起多个块。Encoder 最后输出 `z_mean` 和 `z_logvar`；Decoder 最后输出 `x_mean` 和 `x_logvar`。

- **VAE 损失**：重建高斯负对数似然加上 `beta * KL(q(z|x)||N(0,I))`；对每个样本的像素或潜变量维度求和，再对 batch 平均。

VAE 的 ResidualBlock 通过归一化、卷积、激活和残差连接提取特征；零初始化的残差分支让网络刚开始时不必立刻大幅改变输入。AttnBlock 将空间位置展平为 token，做归一化、多头注意力和 MLP，再还原空间形状，使模型同时具备卷积的局部特征提取和注意力的全局关系建模能力。EncoderBlock / DecoderBlock 在这些基础模块上增加下采样或上采样：编码器逐步压缩空间分辨率，解码器逐步恢复分辨率。实现这些结构时，通道数与高宽变化必须和相邻块匹配；它们帮助理解“压缩表示”不是抽象概念，而是由具体的空间尺度变化实现。

Encoder 输出 `z_mean` 和 `z_logvar`，表示近似后验分布 `q(z|x)` 的参数；Decoder 输出重建分布的参数（例如 `x_mean`、`x_logvar`）。因此 VAE 并非只把图像映射成一个确定的编码，而是学习潜变量的概率分布，并通过重参数化 `z=z_mean+exp(0.5*z_logvar)*epsilon` 保持采样过程可反向传播。VAE 损失由重建高斯负对数似然和 `beta * KL(q(z|x)||N(0,I))` 组成：前者要求解码结果保留输入信息，后者使潜空间接近便于采样的标准正态分布。按图像像素和潜变量维度求和后，loss 数值达到数百甚至更高并不自动意味着错误；应结合归约方式、`beta`、重建结果和训练趋势判断。完成这一部分，能理解 latent diffusion 为什么可行：先把图像压缩成具有结构的潜变量，再在较小表示上生成。


### Part 5：Latent CFG

- **LatentCFGTrainer 损失**：从 MNIST 采样 `(x,y)`，在 `torch.no_grad()` 中编码并采样潜变量 `z`；随后执行与 CFG 相同的标签丢弃、路径采样和向量场回归。

- **生成过程**：从潜空间简单分布采样，用 CFG 引导向量场积分得到潜变量，再交给 VAE decoder 解码。

LatentCFGTrainer 先从图像和标签得到 VAE 潜变量，再在潜空间执行与 Part 2 相同的标签随机丢弃、概率路径采样和向量场回归。通常预训练 VAE 在此阶段被冻结，因此编码应放在 `torch.no_grad()` 中；这既避免无用梯度，也减少显存占用。训练对象由此变为 latent DiT，而不是再次训练 VAE。采样时，CFG 在潜空间引导向量场，生成完成后再由 VAE Decoder 把潜变量还原为图像。这个最后的组合题检查你是否真正掌握各组件的职责边界：VAE 负责表示与解码，DiT 负责潜空间动力学，CFG 负责调整条件引导强度。


### total

1. **从目标函数到训练循环**：自己实现 CFG loss，能把概率路径、条件标签、null label 和模型预测目标对应起来，理解模型究竟在学什么。

2. **从张量形状到网络结构**：实现 patchify、注意力和 depatchify，会练习追踪 `(b,c,h,w)` 与 `(b,n,d)` 的转换，避免只会调用模型却不清楚数据如何流动。

3. **理解条件信息如何进入模型**：类别 embedding、时间 Fourier 特征和 AdaLN 展示了不同条件如何影响每一层的计算。

4. **理解稳定训练设计**：残差连接、零初始化、归一化、`no_grad()` 与方差参数化，都对应训练稳定性、梯度范围或内存开销等实际问题。

5. **理解生成系统的模块边界**：VAE 负责压缩和重建，latent DiT 负责建模潜变量分布；分开检查它们能更快定位图像异常来自编码、生成还是解码。

6. **培养调试能力**：`NotImplementedError` 提醒检查实际执行到的占位函数；类型与形状错误则训练从 traceback 定位模块边界，再核对输入输出契约。


| 部分 / 问题                                     | 实现要点回顾                                                                                                                        | 这题在训练什么理解                                                                                |     |                                      |                                                      |
| ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | --- | ------------------------------------ | ---------------------------------------------------- |
| **Part 2：CFG 训练损失**                         | 采样图像 `z` 和标签 `y`；以概率 `eta` 将标签替换成 null label；采样 `t`、`x_t`，回归模型向量场和 `conditional_vector_field(x_t, z, t)` 的平方误差。               | 把条件 Flow Matching 的数学期望落实为 Monte Carlo batch；理解训练时随机丢标签，才能让同一个模型学到有条件和无条件预测。             |     |                                      |                                                      |
| **Part 2：MLP 条件向量场**                        | 通过 `nn.Embedding` 编码标签，将标签 embedding、时间和输入 `x` 拼接后输入 MLP；输入维数为 `dim + class_dim + 1`。                                         | 练习把类别条件转成网络可用的连续表示，并核对输入张量形状和网络层维度。                                                      |     |                                      |                                                      |
| **Part 3：FourierEncoder**                   | 计算 `2*pi*weights*t`，拼接正弦和余弦特征。                                                                                                | 理解扩散时间是模型的重要条件；多频率特征让网络更容易表示随时间变化的行为。                                                    |     |                                      |                                                      |
| **Part 3：Patchifier / Depatchifier**        | Patchifier 用卷积（`kernel_size=stride=patch_size`）把图像切成 token，输出 `(b, n, dim)`；Depatchifier 将 token 映射回 patch 特征，重排为空间特征图并卷积成图像。 | 学会在图像张量 `(b,c,h,w)` 与 Transformer 序列 `(b,n,d)` 之间转换，也能理解 patch 大小对 token 数量、计算量和空间细节的影响。 |     |                                      |                                                      |
| **Part 3：MHA**                              | 线性层生成 Q/K/V，拆分 heads，计算 `softmax(QKᵀ/sqrt(head_dim))V`，再合并 heads 并投影。                                                         | 从公式理解注意力如何让不同图像 patch 交换信息；通过维度检查掌握 `dim` 必须能被 `heads` 整除。                               |     |                                      |                                                      |
| **Part 3：DiT layer / DiffusionTransformer** | DiT layer 用条件向量产生注意力、前馈两支的 AdaLN shift、scale、gate，并通过残差连接；末层零初始化。Transformer 添加可学习位置编码，再堆叠多个层。                                | 理解时间和类别条件如何调制网络内部计算；位置编码保留 token 空间位置，残差和零初始化有助于深层网络稳定训练。                                |     |                                      |                                                      |
| **Part 3：完整 DiT flow model**                | 时间经 FourierEncoder、类别经 embedding 编码，两者相加作为条件；图像依次经过 Patchifier、DiT 和 Depatchifier，输出向量场。                                      | 将独立模块串成完整的数据流，检查模型输入、条件和输出是否符合 Flow Matching 的接口。                                        |     |                                      |                                                      |
| **Part 4：ResidualBlock**                    | GroupNorm → 3×3 卷积 → 激活 → 1×1 卷积 → 输入残差；第二个卷积零初始化。                                                                            | 练习 CNN 中局部特征提取和残差学习；零初始化让残差分支起始时近似恒等映射，帮助训练稳定。                                           |     |                                      |                                                      |
| **Part 4：AttnBlock**                        | 把空间展平成 token，依次做 LayerNorm + MHA + 残差、LayerNorm + MLP + 残差，再还原空间维。                                                            | 理解卷积的局部建模与注意力的全局建模互补；同时练习图像空间与 token 序列的变换。                                              |     |                                      |                                                      |
| **Part 4：EncoderBlock / DecoderBlock**      | 两个残差块加注意力；EncoderBlock 可选步幅 2 卷积下采样，DecoderBlock 可选上采样后卷积调整通道。                                                                | 看懂编码器缩小空间、增加语义压缩，解码器反向恢复空间分辨率的结构设计。                                                      |     |                                      |                                                      |
| **Part 4：Encoder / Decoder**                | Encoder 串接编码块，输出 `z_mean` 和 `z_logvar`；Decoder 串接解码块，输出 `x_mean` 和 `x_logvar`。                                                | 把小模块组合成概率编码器和解码器，理解 VAE 不只预测一个确定的潜变量，而是学习分布参数。                                           |     |                                      |                                                      |
| **Part 4：VAE 损失**                           | 重建高斯负对数似然加上 `$beta * KL(q(z \| x) \| N(0,I))$`；按样本对像素/潜变量维求和，再对 batch 求平均。 | 理解 VAE 的两种目标：重建质量与潜空间正则化之间的权衡；也能解释 loss 为什么可能是数百或数千。 |
| **Part 5：LatentCFGTrainer**                 | 从 MNIST 采样 `(x,y)`，在 `torch.no_grad()` 下编码并采样潜变量 `z`；随后执行 CFG 标签丢弃、潜空间路径采样和向量场回归。生成时由 latent DiT 采样潜变量，再交给 VAE 解码。            | 把前面学到的 CFG、DiT、VAE 组合起来；`no_grad()` 体现冻结已训练 VAE、只训练潜空间生成器的模块边界。                          |     |                                      |                                                      |

## 9. 实验调试记录与常见问题

### 9.1 `conv2d()` 收到 `SiLU`，而不是 Tensor

通常是把激活函数类保存为 `self.act = act`，后面调用 `self.act(x)` 时实际上创建了一个 `SiLU` 模块。应在初始化时实例化激活层，例如 `self.act = act()`，并按 `x = self.act(x)` 使用。

### 9.2 Notebook 单元执行顺序

Notebook 内存中的类定义可能与磁盘上当前看到的代码不同。修改类后如果仍出现旧错误，先重新运行相关定义单元；遇到状态混乱时，重启 kernel 后从上到下执行。

### 9.3 CPU 使用率很高

MNIST 采样器若逐项从数据集取样并做变换，CPU 可能在为 GPU 准备 batch，导致 CPU 使用率很高。需要确认 `torch.cuda.is_available()`、模型参数所在设备以及 GPU 利用率；若模型设备是 CPU，则训练速度会显著变慢。CPU 高占用本身不等于程序错误。

### 9.4 图像看起来异常

排查顺序：

1. 先看 VAE 的重建图像是否合理。若重建就异常，优先检查 VAE 训练、预处理和归一化。
2. 确认潜空间尺寸、通道数与 DiT 输入输出及概率路径一致。
3. 确认 latent DiT 训练步数足够；1000 步、batch 64、学习率 `1e-4` 是小规模试跑，不等同于实验注释中的长时间训练设置。
4. 检查采样时间是否避开路径在终点处的不稳定区域，并确认图像显示使用了适当的归一化。
5. 改善样本质量时，逐步增加训练步数/批量或调整学习率；不要只根据单个 loss 数值判断是否异常。

## 10. 快速回顾清单

- [ ] `MNISTSampler.sample()` 返回 `(image, label)`，标签形状为 `(b,)`。
- [ ] CFG 的 null label 是独立类别（MNIST 常用 `10`），embedding 总类别数为 `11`。
- [ ] 区分训练标签丢弃率 $\eta$ 与采样引导强度 $w$。
- [ ] CFG 推理公式为 $(1-w)u_{\mathrm{null}}+wu_{\mathrm{cond}}$。
- [ ] Patchifier 输出 token 数为 $(H/p)(W/p)$，DiT 的 `dim` 必须能被注意力头数整除。
- [ ] 时间编码的输入形状与 FourierEncoder 要求一致。
- [ ] VAE 的 encode 在 latent DiT 训练时放在 `no_grad()` 中。
- [ ] latent probability path 的 shape 与 VAE 潜变量一致。
- [ ] 生成质量需要看样本图；loss 下降仅说明训练目标在改善，不保证样本一定可用。
- [ ] 在 Notebook 中修改定义后，重新运行类定义和实例化单元。

## 11. 总结

本实验先用标签丢弃训练出同时具备有条件与无条件能力的向量场，再通过 CFG 在生成时强化类别控制；DiT 负责建模图像 token 之间的关系，VAE 则把高维像素生成转换为较低维潜空间生成。