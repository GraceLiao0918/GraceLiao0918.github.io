---
title: 🫣环境设置
tags:
  - tool
---

# node.js

下载：https://nodejs.org/zh-cn

版本：`node -v v24.20.0  npm -v 11.19.0`

##### PowerShell中npm命令无法运行的问题

在PowerShell中使用npm可能会出现问题，这通常是因为PowerShell的执行策略限制了运行脚本的能力。

> 更改 PowerShell 执行策略

您可以尝试将PowerShell的执行策略更改为允许脚本运行。在#管理员权限 下的PowerShell窗口中运行以下命令：

`Set-ExecutionPolicy RemoteSigned`

然后选择 Y 以确认更改。


# git

下载：https://git-scm.com/install/windows

版本：
`git -v git version 2.55.0.windows.5`

使用：git Bash here


# Anaconda

下载：[Download Success - post download | Anaconda](https://www.anaconda.com/download-success)

版本：2026-07

是否安装成功？anacanda prompt 中（base）

创建环境：conda create -n pytorchLearn python=3.7

激活环境：conda activate pytorchLearn

# Pytorch

下载：[Get Started](https://pytorch.org/get-started/locally/)

版本：2.14.0 CUDA_13.0

在anaconda中安装：pip3 install torch torchvision --index-url https://download.pytorch.org/whl/cu130

查看本地电脑CUDA版本：nvidia-smi

查看anaconda的环境包：pip list

检查pytorch是否可用：
- python
- import torch
- torch.cuda.is_available()   Ture

![[Pasted image 20260914143813.png]]

