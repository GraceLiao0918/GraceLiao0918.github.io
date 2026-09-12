---
title: 情感计算+大模型综述
tags:
  - 多模态大模型
  - 情感计算
  - 强化学习
---
# 情感计算+大模型：从判别式情感识别迈向生成式情感理解
#情感计算 #多模态大模型 #MLLM #情感识别 #AffectGPT #研究综述

> [!abstract] 摘要
> 多模态大模型助力情感理解，探索生成式与可解释性方法，推动领域发展。

将情感计算与多模态大模型相结合是目前的研究热点。本文总结了我们近期在该领域的一些尝试，希望借助多模态大模型丰富的词表，以及多模态感知能力，将判别式情感识别，向细粒度、可解释生成式情感理解过渡，并整理了我们近期在该领域的一些尝试，包括 **EMER**(Arxiv24)、**OV-MER**(ICML25)、**AffectGPT**(ICML25)、**EmoPrefer**(Arxiv25) 和 **AffectGPT-R1**(Arxiv25)，希望能给情感计算社区提供一些参考。
### 表 1 不同工作之间的关系

| 工作           | 说明                         |
| ------------ | -------------------------- |
| EMER         | 提出可解释性多模态情感推理任务            |
| OV-MER       | EMER 任务的子问题                |
| AffectGPT    | EMER/OV-MER 数据构建方案以及模型框架改进 |
| EmoPrefer    | EMER 任务的评价方法               |
| AffectGPT-R1 | EMER/OV-MER 训练策略改进         |

## 01 摘要

情绪与认知、决策和行为密切关联，在人工智能领域发挥着关键作用。情绪表征方法旨在将复杂的人类情绪映射为可量化的数值。当前情绪表征领域主要存在两种范式：**基本情感表示**与**维度情感表示**。
### 基本情感表示

基本情感表示基于心理学理论，将人类情绪划分为离散标签。例如，Ekman 指出所有文化中都存在一组普适的情感状态，包括愤怒、厌恶、恐惧、快乐、悲伤和惊讶。然而，人类情绪非常复杂，远不止六种基本情感，将情绪空间限制于基本类别必然会忽略一些细微情绪。

> [!warning] 基本情感表示的局限
> Mapping the rich emotional states of humans to a small number of basic labels inevitably leads to **inaccurate emotional descriptions**.

### 维度情感表示

不同于基本情感，维度情感表示将人类情绪建模为连续多维空间中的一个点，能够建模细粒度情感。例如，Russell 提出 PAD 模型，采用**效价度**(Valence) 描述情绪积极程度，**唤醒度**(Arousal) 描述情绪兴奋水平，**支配度**(Dominance) 描述人类对情绪的控制力度。但是，维度表示相比于离散表示更为抽象，与人类对于情感的直观感受不一致，导致维度情感在下游任务中应用较为受限。

> [!warning] 维度情感表示的局限
> It requires professional knowledge and **does not align with human subjective emotional experiences**.

随着多模态大模型（MLLMs）的发展，情感理解得以突破传统判别式框架的局限，转向更具表达力的生成式框架。借助大规模词表，MLLMs 能够生成超越基础情感的多样化、细粒度情感类别，为准确的情感理解提供了新的可能。此外，我们可以结合 MLLMs 的多模态感知能力，利用多模态线索与分析过程，具象化人类情感的产生过程，增强情感表示的可解释性。

## 02 EMER：可解释情感推理

可解释性多模态情感推理（EMER）希望借助多模态大模型，用自然语言描述人类情感产生过程，增强情感表示的准确性与可靠性。

![[assets/02Emolearning/Pasted image 20260910164331.png]]
> 图 2 可解释性多模态情感推理（EMER）
>
> - **(a) Diversity**：模型可从面部表情、语气语调等多模态线索推理出 "mocking / sarcastic tone" 等细粒度情感。
> - **(b) Coexistence**：模型可同时识别 "surprised facial expression"、"nervousness, anxiety, or unease"、"unhappy and angry" 等共存情感。

### 数据构建

我们招聘多名标注人员从以下方面标注情感标签以及产生依据，包括面部表情、身体动作、语气语调、文本内容、视频内容和环境等多模态线索。然后，我们利用 LLM 汇总多名标注人员的情感线索。最终，我们手动核对推理过程，最终得到了 **332 条 EMER 数据集**。

![[02Emolearning/Pasted image 20260910163127.png]]

> 图 3 EMER 数据集构建过程：Unlabeled videos → Pre-label → Two-round Checks → acoustic clue / visual clue / subtitle / lexical clue → EMER (Multi)

### 评价指标

我们提出了两种评测指标：基于 ChatGPT 的自动评价结果，以及人工评测结果。

- **自动评价指标**：对情感重叠度、情感线索重叠度、以及模态完备性进行评估。
- **人工评测**：聘请多名标注人员对情感推理过程的合理性进行打分。

> [!note] EMER 的局限
> EMER 为情感表示提供了一个新的思路，但仍存在很多问题有待解决：
> 1. **数据集规模难扩大**：构建过程中需要大量人工标注。
> 2. **评价依赖真实描述**：基于 LLM 的评价需要依赖于真实情感描述。然而，情感本身与多种人类行为紧密相关（面部表情、(微) 姿态、头部动作、手部动作以及语音语调等），生成全面且准确描述情感状态的文本本身具有很大挑战，而不准确或不可靠的真实描述也会导致评估结果不可靠。

## 03 OV-MER：开放词汇情感识别（ICML25）

现有方法通常依赖于有限的基本情感标签，这些传统且过于简单的情感类别未能捕捉到人类情感的细微差别。因此，我们提出了一项新任务——**开放词汇情感识别（OV-MER）**。

它涵盖更广泛的情感类别，扩展了标签空间，允许预测任意数量、任意类别的情感状态。针对这一任务，我们提供了一套解决方案，包括：

- 基于大型语言模型和人类协作注释的数据库构建策略；
- 相应的评估指标；
- 基准结果。

OV-MER 是 EMER 的子问题，将研究重心从情感描述转向为更容易标注的情感标签，降低了不同模型性能评估难度。

![[assets/02Emolearning/Pasted image 20260910164158.png]]

> 图 4 开放词汇情感识别（OV-MER）
>
> | 对比维度 | One-hot MER | Multi-label MER | Open-vocabulary MER |
> | --- | --- | --- | --- |
> | Label Space | Basic Emotion（固定小集合） | Basic Emotion（固定小集合） | Any Emotion（开放） |
> | Label Number | One Label | Multiple but Limited | Any Number |
> | Annotation Manner | Human-only | Human-only | Human-LLM Cooperation |

与 EMER 相比，OV-MER 绕过对情感描述的评估，转而聚焦于情感标签。相比于情感描述，情感标签可以采用多人众包标注，更容易获取可靠的标签。因此，OV-MER 可以认为是**简化的 EMER 问题**。


## 04 AffectGPT：描述性情感识别（ICML25）

MLLMs 在情感理解方面仍存在诸多局限性。主要原因在于：

1. 缺乏包含描述性情感标注的大规模数据集；
2. 缺乏以多模态为核心的框架来最大化 MLLMs 的情感理解潜力。

针对这些问题，我们构建了一个面向生成式情感理解框架的数据集（**MER-Caption**）和模型架构（**AffectGPT**）。借助人类先验知识引导自动化标注流程，我们构建了迄今为止规模最大的描述性情感数据集。同时，我们提出了 AffectGPT 框架，该模型采用**预融合操作**来增强多模态信息的整合能力。本文从数据集构建与模型结构设计两个维度出发，试图解决 EMER 问题。

### 数据构建

现有的描述性情感数据集构建方法可归纳为三类：

> [!example] 三种数据构建策略
> 1. **模型驱动**：自动标注方法，但由于缺乏人工校验环节，标注质量可能存在不足。
> 2. **人工主导**：目前构建高质量情感数据集的主流方法。然而，纯手工标注在规模化扩展数据集时成本高，且标注者往往倾向于主要线索，导致生成的情感描述趋于简单或不完整。
> 3. **人机协同**：模型先提供预标注线索，再由人工进行多轮审核，本质上是一种 "以人为主导、模型为辅助" 的数据标注流程。尽管这种方法能产生更全面的描述，但其高昂的成本和低扩展性限制了大规模数据集标注。

为在标注质量和数据规模之间取得平衡，本文提出一种 **"以模型为主导、人工为辅助"** 的标注策略，结合模型的众包标注与人类先验知识实现自动化数据标注。该策略充分发挥了人类先验知识的指导作用，用于指导描述生成及样本筛选，最终实现了对无标注数据的自动化标注。

基于这一策略，我们构建了 MER-Caption 数据集：

- **11.5 万条**粗标注样本；
- **3.1 万条**细标注样本。

![[assets/02Emolearning/Pasted image 20260910164338.png]]

> 图 7 MER-Caption 数据集构建流程：
> **Model-led Human-Assisted Description Generation**（Text/Audio/Video → ALLM + VLLM → GPT Merge）→ MER-Caption 115k → **Model-led Human-Assisted Sample Filtering**（Low-level filtering + High-level filtering）→ MER-Caption+ 31k

### 模型结构

与文本大语言模型不同，MLLM 的关键挑战在于实现**多模态感知**——即赋予模型"视觉"与"听觉"能力。当前主流方案采用 **"模态编码器 + 投影层"** 的架构，通过专家网络提取各模态特征后，经投影层映射至语言模型的统一语义空间。

尽管该架构已取得显著进展，但跨模态交互完全依赖 LLM 内部处理，难以充分利用多模态融合信息。因此，我们提出**预融合操作（Pre-fusion）**，将跨模态交互迁移至 LLM 外部执行，增强多模态特征整合能力。

![[assets/02Emolearning/Pasted image 20260910164349.png]]

> 图 8 AffectGPT 模型框架：
> - Audio LLM (ALLM)
> - Video LLM (VLLM)
> - Audio-Video LLM (AV-LLM)
> - **AffectGPT (Ours)**：引入 Pre-fusion & Projector，在 LLM 外部完成音视频特征融合

---

## 05 EmoPrefer：情感偏好学习（Arxiv25）

EMER 旨在利用自然语言来描述人类的情感状态。然而，评估此类开放式描述质量仍然是一项具有挑战性的任务。

现有的几种评估策略：

1. **基于真实描述 + 大模型相似度**：利用真实情感描述，并借助大模型（如 OpenAI GPT 或 Google Gemini）来衡量预测描述与真实描述之间的相似度。但情感本身与多种人类行为紧密相关，生成全面且准确描述情感状态的文本本身具有很大挑战，而不准确或不可靠的真实描述也会导致评估结果失真。
2. **偏好驱动（Preference-driven）**：放弃那些成本高昂且往往不完整的真实描述，转而将"设计用于衡量与真实描述语义相似度的指标"这一复杂问题，重新表述为一个更易处理的"学习人类偏好"的问题。但这种方法需要对多个模型在多个样本上的每一对组合进行偏好标注，成本较大。

![[assets/02Emolearning/Pasted image 20260910164420.png]]

> 图 9 EMER 任务的评估策略
> - **(a) Description-based**：Ground Truth Description ↔ Predicted Description，通过 LLM-based Metrics 衡量。
> - **(b) Preference-driven**：Human Annotator 对 Predicted Description 1/2 打偏好。
> - **(c) EmoPrefer**：用 Judge MLLM 替代人工 Annotator，自动判定偏好。

因此，我们能否利用多模态大模型实现情感偏好的自动解码，从而为描述性多模态情感识别提供一种更具成本效益的评估策略？

为了回答这一问题，我们提出了 **EmoPrefer**——这是首个探索多模态大模型在情感偏好预测方面潜力的研究工作。我们构建了：

- **EmoPrefer-Data**：人类标注的偏好数据；
- **EmoPrefer-Bench**：基于 MLLM 的评测基准。

我们对不同多模态大模型及其提示策略进行了全面评估，并进一步探索了如何提升模型与人类偏好的一致性。

![[assets/02Emolearning/Pasted image 20260910164426.png]]

> 图 10 EmoPrefer 框图：
> 1. **EmoPrefer-Data (Human)**：从 Emotion Dataset 1/2 中选取 Overlapping Selection x，人工对 Des d₁ / Des d₂ 打星并给出偏好，如 $p_{ij} = \{d_1 > d_2\}$。
> 2. **EmoPrefer-Bench (Judge MLLM)**：Model Pool（多种 MLLM）对同一输入 x 生成 pred 与 des，形成 S1–S4 测试场景。
> 3. **Evaluation Metric**：以人工偏好为 true，衡量 Judge MLLM 的 Recognition Performance；并通过 swap d₁/d₂ 检验 Swap Consistency。

从 EMER 任务评价困难的角度出发，EmoPrefer 将"设计用于衡量与真实描述语义相似度的指标"这一复杂问题，重新表述为一个更易处理的"学习人类偏好"的问题，并探索了自动化情感偏好标注的可能性。


## 06 AffectGPT-R1：推理增强情感理解

OV-MER 依赖于 EW-based 评估指标，然而 EW-based 评估指标无法通过梯度反传进行优化。因此，我们提出了 **AffectGPT-R1**，将 EW-based 评估指标作为奖励函数的一部分，并采用 **GRPO** 对奖励函数进行优化。我们发现 AffectGPT-R1 在基本情感识别与开放词汇情感识别任务上取得了性能提升。

![[assets/02Emolearning/Pasted image 20260910164651.png]]

> 图 11 AffectGPT-R1 总结
>
> **Stage 1: Cold Start**
> 输入 User Message，要求模型识别角色的所有可能情绪状态，输出思考过程 `<think>...</think>` 与最终答案。
>
> **Stage 2: RL (Reward Function + GRPO Optimizer)**
>
> 格式奖励：
> $$R_{format}(o|v,q) = \begin{cases} 1, & o \text{ strictly follows the required format} \\ 0, & \text{otherwise} \end{cases}$$
>
> 准确率奖励：
> $$R_{accuracy}(o, y|v, q) = \mathrm{EW}(o \to a, y)$$
>
> 综合奖励：
> $$R(o, y|v, q) = R_{accuracy}(o, y|v, q) + \beta\, R_{format}(o|v, q)$$
>
> **Framework**：基于 AffectGPT，对比 (a) w/o thinking 与 (b) w/ thinking 两种推理模式。
>
> **Performance（Table 4: Necessity of thinking process）**
>
| Model | OV-MERD+ |
| --- | --- |
| AffectGPT | 62.52 |
| AffectGPT-R1 (w/ thinking) | 66.49 |
| AffectGPT-R1 (w/o thinking) | 68.39 |


## 总结

> [!tip] 研究脉络
> 从 **EMER**（可解释情感推理任务提出）→ **OV-MER**（子问题：开放词汇标签）→ **AffectGPT**（数据构建 + 预融合模型框架）→ **EmoPrefer**（偏好学习式评测）→ **AffectGPT-R1**（GRPO 强化推理训练），我们沿"任务定义 → 数据 → 模型 → 评测 → 训练策略"的链路，逐步推动情感计算从**判别式情感识别**迈向**生成式情感理解**。


