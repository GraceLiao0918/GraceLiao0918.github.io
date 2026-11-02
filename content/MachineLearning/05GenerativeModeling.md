---
title: 😱生成式模型
tags:
  - GenerativeModeling
---
# 生成式模型

## 生成式模型的主要流派

[一图读懂生成式模型 (Generative Modeling) 的主要流派 - 知乎](https://zhuanlan.zhihu.com/p/1905427654582183119)

[2504.21853](https://arxiv.org/pdf/2504.21853)



### 建模复杂的数据分布

- 生成式模型的终极目标，是建模真实世界数据的**复杂联合分布 (Complex Joint Distribution)**。简单来说，就是让模型理解数据的内在模式和结构，从而能够“创造”出与真实数据相似的新数据。
- 想象一下，我们要生成猫的图片。真实世界中猫的图片千姿百态，它们的颜色、姿势、背景、光照等等构成了极其复杂的数据分布。生成式模型的任务就是学习这个分布，然后从中“采样”出新的、看起来像猫的图片。
- 然而，直接学习如此复杂的分布极为困难。当前主流的技术路径可以分为两个主要大类：

### 分布分解建模 (Distribution Factorization Modeling) 

这种策略的核心思想，是利用概率论中的链式法则，将复杂的高维联合分布 $p(x)$ 分解为一系列条件概率的乘积：$p(x)=p(x1)p(x2∣x1)p(x3∣x1,x2)...$

这种方法的典型代表就是**自回归模型 (Autoregressive Models, AR)**。

- 什么是自回归？就像我们说话或写作时逐字逐句地进行一样，自回归模型生成数据也是序列化的过程。它根据已经生成的部分，来预测下一个“单元”（例如下一个像素、下一个词元/Token）
    - _注：在恺明的课程《6.S978 Deep Generative Models》中提到：“In general, autoregression is a way of modeling joint distribution by a product of conditional distributions.” 这是一个广义定义。若严格按此定义，扩散模型某种意义上也可视为一种特殊的条件分布建模。但在通常语境下，我们区分 AR 和 Diffusion 的一个实用角度是看初始采样来源：AR 模型通常从数据本身的分布中开始采样第一个元素；而 Diffusion 类模型则从一个预设的简单先验分布（如高斯分布）开始采样。_
- 主要类型：
    - **NTP (Next Token Prediction):** 最经典的方式，广泛应用于大型语言模型（例如 GPT 系列）、早期的图像模型（例如 DALL·E 1）、视频生成模型（例如 VideoGPT）以及原生统一的多模态模型（例如 EMU3）
    - **NSTP (Next Set-of-Tokens Prediction):** 一次预测或生成一组词元，常见于图像生成（例如 MaskGIT）
    - NSP (Next Scale Prediction): 在不同尺度或分辨率上进行预测，例如图像生成中的VAR
    - 补充说明：NTP/NSTP 通常将中间生成的词元直接作为最终输出的一部分；而 VAR的中间预测结果更多是作为后续步骤的条件，不直接构成最终输出。

### 分布映射建模 (Distribution Mapping Modeling) 

与因子分解不同，该策略试图学习一个映射函数 (Mapping Function) $G$，将一个容易采样的简单分布 $p_Z(z)$（通常是高斯噪声）直接或间接地变换到目标数据的复杂分布 $p_X(x)$ 上，即 $x=G(z)$
- 主要分支：
    - 直接映射建模 (Direct Mapping Modeling): 学习一个（通常是单步的）直接映射函数。
        - **生成对抗网络 (GAN - Generative Adversarial Networks):** 通过生成器 G 和判别器 D 的相互博弈来学习映射。G 努力生成以假乱真的数据，D 则努力区分真实数据与 G 生成的数据。
        - **变分自编码器 (VAE - Variational Autoencoders):** 包含一个编码器将数据 $x$ 映射到潜在空间 $z$ ，和一个解码器从 $x$ 重建数据 $\hat{x}$ 。通过优化重建损失和对潜在空间施加先验分布（如高斯分布）约束，解码器部分具备了从先验分布采样 $z$ 并生成 $x$ 的能力。
    - **概率路径建模 (Probability Path Modeling):** 这类方法不寻求一步到位的映射，而是构建一个连续或离散的演化路径，将样本从简单的初始分布（如噪声）逐渐转化为目标分布。某种程度来说，概率路径建模也是一种条件分布建模，也有分布分解建模的思想，它是 Distribution Factorization Modeling 和 Distribution Mapping Modeling的混合。
        - **扩散模型 (Diffusion Models)** 和 **流匹配模型 (Flow Matching Models)** 。**注:** Diffusion Model 和Flow Matching Models 可以被统一到一个框架内。我们经常用“Diffusion” 用指代这类基于概率路径的方法，即包含Flow Matching 和Diffusion


### 混合建模 (Hybrid Modeling - AR + Diffusion)

最近大家尝试将自回归模型和扩散模型结合起来，这催生了混合模型 (Hybrid Models)，特别是 **AR + Diffusion** 的组合。
- 为什么结合？
    - **优势互补**: AR 模型擅长捕捉序列依赖关系，而 Diffusion 模型在生成高保真度的细节和纹理方面表现卓越（尤其在视觉内容上）。对于图像、视频等多模态内容生成，Diffusion 的地位目前难以被纯 AR 模型取代
    - **统一多模态:** 随着对原生统一多模态模型（如能同时理解和生成文本、图像、视频等）的追求，需要有效融合以 AR 为主流的文本建模范式和以 Diffusion 为主流的视觉建模范式。
- **四种主要结合方式：** (它们是光谱上连续的点，从AR占主导到Diffusion占主导)**:**
    1. **共享参数的 AR 和 Diffusion（AR and Diffusion with shared parameters）[代表工作: Transfusion]:** 使用统一的架构（如 AR Transformer）和同一套参数同时处理自回归和扩散两个任务。他们只是形式上统一了。由于是两类很不同的任务在一起，其最终的效果可能会互相打架。
    2. AR backbone + Diffusion loss/sampler [代表工作: MAR]: 以AR为主体，引入diffusion loss。一方面，它解决了连续 Token 在AR结构中的训练问题；另一方面，正如在MAR中说明的那样：“AR models interdependency, while Diffusion models per-token distribution”，即利用 AR 建模元素间的依赖关系，利用（通常轻量的）Diffusion Head 对每个元素的具体分布进行建模。这类方法和下面的以EMU2为代表的方法很不相同，不同点在于：生成一个条件 (Condition) -> Diffusion 采样得到输出 Token -> 将该 Token 作为下一个输入 -> 生成下一个条件 -> Diffusion 采样... 如此循环
    3. 解耦的 AR 和 Diffusion 阶段（Decoupled AR and diffusion stages）[代表工作： EMU2]: 采用两阶段方法：先用 AR 模型生成描述目标输出的条件表示（例如一系列 embedding 或 条件 token），然后将这些条件表示一次性输入给一个强大的 Diffusion 模型，由 Diffusion 模型负责最终的解码生成
    4. 自回归扩散模型（Autoregressive diffusion models）[代表工作：Diffusion Forcing, CausalFusion]: 将自回归的生成方式融入 Diffusion 模型本身。这类方法本质上还是Diffusion。它和第二类方法 AR backbone + Diffusion loss/sampler 还有一个不同之处在于，它建模帧与帧之间的关系还是在Diffusion内部，而不采用一个单独的AR模型来建模。

