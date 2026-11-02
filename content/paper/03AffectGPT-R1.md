---
title: AffectGPT-R1
tags:
  - 强化学习
  - 情感计算
  - 情感识别
---



# AffectGPT-R1: Leveraging Reinforcement Learning for Open-Vocabulary  Multimodal Emotion Recognition

本文提出了一种基于强化学习的开放词汇情感识别框架AffectGPT-R1，旨在解决传统方法中目标函数与评估指标不一致的问题。开放词汇多模态情感识别（OV-MER）通过生成模型（如大语言模型）实现更细粒度、更接近人类理解的情绪识别。然而，已有方法如AffectGPT主要依赖token-level损失进行训练，与基于情绪轮（EW）的评价指标相关性较低，且这些指标不可微分，难以直接优化。为此，AffectGPT-R1将EW-based指标作为奖励函数，采用Group Relative Policy Optimization（GRPO）策略进行强化学习优化，从而实现对这些指标的直接优化。实验结果显示，AffectGPT-R1在OV-MER任务上取得了显著提升。论文还对比了R1-Omni框架，指出AffectGPT-R1在任务目标、训练数据规模及开源程度上的改进。


|问题|内容|
|---|---|
|**Problem**|**解决开放词汇多模态情感识别（OV-MER）中训练目标与评价指标不一致的问题。** 传统 AffectGPT 主要采用 token-level loss 进行监督训练，即让模型生成的文本尽可能接近标注文本；但 OV-MER 最终采用基于 **Emotion Wheel（EW）** 的指标评价情感预测结果。二者优化目标并不一致，而且 EW-based 指标不可微，无法直接用于传统反向传播。因此，AffectGPT-R1 将 EW-based 情感评价指标转化为 **Reward**，通过强化学习直接优化最终评价目标。|
|**State**|**多模态视频信息 + 用户查询（Query）。** 输入的视频 vv 同时包含 **视觉（Visual）、声学（Acoustic）和语言（Lexical）** 信息，例如面部表情、声音特征以及说话内容；同时结合用户提出的情感识别指令 qq。因此可以表示为 s=(v,q)s=(v,q)。需要注意：这里的 State 并不是传统强化学习中不断变化的环境状态，而更接近于模型进行一次情感识别任务时接收到的 **多模态上下文/Observation**。|
|**Action**|**大语言模型生成的完整回答。** 给定一个多模态输入后，AffectGPT-R1 生成结构化的 `<think>...</think><answer>...</answer>` 内容。GRPO 将模型生成的完整序列作为优化对象，而 Reward 中主要从 `<answer>` 和 `</answer>` 之间提取最终情感词/情感描述，并与真实情感标签进行比较。因此可以理解为：**Action = LLM 生成的情感识别回答**。|
|**Reward**|**由 EW-based Accuracy Reward 和 Format Reward 两部分组成。** 总奖励可以表示为 R=Raccuracy+βRformatR=R_{\text{accuracy}}+\beta R_{\text{format}}。其中，RaccuracyR_{\text{accuracy}} 根据模型预测的情感与真实情感在 **Emotion Wheel** 上的语义接近程度进行评分，而不是简单进行字符串完全匹配；RformatR_{\text{format}} 用于判断模型是否按照规定格式输出 `<think>...</think><answer>...</answer>`。因此 Reward 的核心目标是让模型**情感预测更准确，同时保持正确的输出格式**。|
|**Algorithm**|**Group Relative Policy Optimization（GRPO）**。对于同一个多模态输入，模型生成一组候选回答 o1,o2,…,oGo_1,o_2,\ldots,o_G，分别计算每个回答的 Reward，然后根据同组回答的平均 Reward 和标准差计算相对 Advantage，再更新语言模型策略。与 PPO 不同，GRPO 不需要额外训练一个 Critic/Value Model，从而降低大语言模型强化学习训练的显存开销。|
|**Dataset/Environment**|**实验采用“Cold Start → RL → Test”的三阶段设置。** ① **Cold Start：MER-Caption+**，包含 31,327 个样本，用于监督训练，使模型获得基本的多模态情感理解和结构化回答能力；② **RL：MER2025-OV**，原始 1,200 个样本，其中 200 个与测试集重叠，因此实际使用 1,000 个样本进行 GRPO 强化学习；③ **Test：OV-MERD+**，包含 532 个样本，用于最终测试。模型架构基于 AffectGPT 的多模态框架，将视觉、音频和语言信息输入大语言模型。严格来说，MER2025-OV 是 **RL training dataset，而不是传统意义上的交互式 Environment**；RL 环境反馈主要由 EW-based Reward 和 Format Reward 构成。|
|**Limitation**|**① RL 数据规模较小：** 实际仅使用 1,000 个 RL 样本，对于多模态大语言模型而言规模有限。**② Reward 设计相对简单：** 主要评价最终情感预测结果和输出格式，没有充分评价视觉、音频等多模态证据是否真正参与推理。**③ 存在 Reward Hacking 风险：** 模型可能针对 EW reward 或格式奖励进行投机优化，而不一定真正提升情感推理能力。**④ `<think>` 不一定代表真实推理过程：** 模型生成合理的情感解释，并不能证明其确实依据正确的视觉、声学和语言证据进行了推理。**⑤ RL 交互性较弱：** 不存在传统 RL 中 st→at→st+1s_t\rightarrow a_t\rightarrow s_{t+1} 的连续状态转移，更接近“一次输入→生成→奖励→策略更新”的 RL 优化过程。**⑥ 任务泛化能力仍需验证：** 论文主要针对 OV-MER 进行实验，对于其他情感相关任务的泛化还有进一步研究空间。**⑦ 消融及超参数探索仍有限：** 不同 Reward、GRPO 设置以及其他 RL 方法对最终性能的独立贡献仍可以进一步研究。|


理解 RL Q-learning-DQN-Policy Gradient-PPO-GRPO
 ↓
做一个文本情感 RL
```
输入："I failed my exam."
模型：happy / sad / angry / neutral
Reward：如果符合真实情感 → +1 否则 → 0
```
 ↓
加入多模态
```
video/audio/text
 ↓
emotion model
 ↓
emotion
```

