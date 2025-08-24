---
title: 🏷️ 双目标局部搜索算法
---

# 基于局部搜索的多目标广告分配问题研究

Bi-Objective Contract Allocation for Guaranteed Delivery Advertising

**作者**：Yan Li, Yundu Huang, Wuyang Mao, Furong Ye, Xiang He, ZHONGLIN ZU, Shaowei Cai

**期刊**：KDD ’24: Proceedings of the 30th ACM SIGKDD Conference on Knowledge Discovery and Data Mining

## 1.合约广告（Guaranteed Delivery Advertising，GD）

- 离线销售阶段：签订合约，基于历史数据与预测系统，预估可用于签订新合约的广告展示量，确定广告订单的需求。
- 在线服务阶段：履行合约，依据实际情况将广告展示量精准分配给各个广告订单。

![[Bi_01.png]]

论文的工作主要集中在已预测出人群流量，并且已签署的合约也已明确的情况下，如何有效地确定当前可供销售的流量库存。具体而言，就是评估和计算平台能够提供给新广告主的最大库存量。

现有研究通常将这两个阶段独立处理。在离线阶段进行合同制定时，往往未充分考虑在线投放阶段的实际投放情况和动态变化，容易出现合约无法完成或库存剩余过多的现象。

论文提出了一种双目标合约分配方法，旨在优化合约广告的资源管理与投放效率：

- 最大化为新到达的广告订单分配的曝光量（即广告资源利用率）
- 平衡广告库存的分配，以提高整体投放的稳定性和可持续性

设计了一套基于局部搜索的算法 BOLS 来对其进行求解：

- 四个阶段：初始化阶段、约束满足阶段、优化阶段、更新解集阶段
- 两种基本的移动操作：单变量移动操作、双变量移动操作

## 2. 问题建模

合约广告分配的二部图模型
![[Bi_02.png]]

- 供应节点: \(S_i \in S\)
- 需求节点: \(D_i \in D\)
- 邻接矩阵\(M=[M_{ij} \in \{0, 1\}]\)

