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

查看现有环境：conda env list

删除环境（非必要）：conda remove -n pytorchLearn --all

> [!danger] danger
> 在pip install ffmpeg，conda环境pytorchLearn下的解释器报错conda 环境内部元数据损坏，出现这个报错后整个pytorchLearn环境无法使用（eg.pip install XXX）,只能修复或者删掉重建（目前环境配置简单，选择了删除pytorchLearn重建pytorchLearning）

# Pytorch

下载：[Get Started](https://pytorch.org/get-started/locally/)

版本：2.14.0 CUDA_13.0

在anaconda中安装：pip3 install torch torchvision --index-url https://download.pytorch.org/whl/cu130

> [!danger] danger
>注意先激活环境，然后执行命令。不要直接pip install torch，会默认下载cpu版本，GPU跑不了
>每次下载之前查看官网的配置和电脑的配置（nvidia-smi），重新复制官网的pip install命令，不要直接复制笔记里的，否则会报错

查看本地电脑CUDA版本：nvidia-smi

查看anaconda的环境包：pip list

检查pytorch是否可用：
- python
- import torch
- torch.cuda.is_available()   Ture

![[Pasted image 20260914143813.png]]




# WSL+Ubuntu

## 前置条件

- 打开“控制面板” → “程序” → “启动或关闭 Windows 功能”
- 勾选“适用于 Linux 的 Windows 子系统”和“虚拟机平台”，点击确定并重启电脑

## 安装

wsl --install -d Ubuntu-24.04 --location D:\Apps\EnvironmentSetup\wsl

如果显示超时等，根据提示进行安装
- wsl.exe --list --online
- wsl --install -d Ubuntu-24.04 --location D:\Apps\EnvironmentSetup\wsl

安装完成后弹窗，手动设置用户名和密码（密码输入无显示，正常输入回车即可）

用户名：liaoziqing
密码：xxxxxxXxx

验证 WSL2 安装成功

- 查看已安装WSL列表及版本 wsl -l -v

### 常用 WSL 管理命令

- 查看已安装WSL列表及版本 wsl -l -v
- 关闭WSL wsl --shutdown
- 重启WSL wsl --shutdown && wsl
- 卸载Ubuntu wsl --unregister Ubuntu
- 查看在线可安装系统 wsl --list --online

# 基于 WSL 在 VS Code 上配置 C/C++ 开发环境

[基于 WSL 在 VS Code 上配置 C/C++ 开发环境在 Windows 环境下，通过 WSL (Windows - 掘金](https://juejin.cn/post/7621082869720612927)

前置条件：安装好WSL+Ubuntu


## 在 Linux 环境检查 C/C++ 编译链

- 检查 GCC 编译器版本 gcc --version 
- 检查 G++ 编译器版本 g++ --version 
- 检查 GDB 调试器版本 gdb --version

如果没配置，执行一下命令进行配置
- 首先更新软件源索引 sudo apt update 
- 安装 build-essential 工具包（包含 gcc, g++, make 等） sudo apt install build-essential gdb

## 插件安装以及连接 WSL

### 插件安装

- 在扩展市场中搜索“C/C++ (Microsoft)”，安装对应扩展
- 下载“Remote - WSL”、“WSL (Microsoft)”这两个扩展，在扩展市场中搜索“wsl”，下载这两个扩展。

创建一个文件夹存源代码 D:\Profile\cProject\wsl，打开文件夹，在上方地址栏中输入“cmd"并回车在该路径打开终端，在终端中输入“wsl”转换为 Linux 环境，在终端中输入命令 code .
### 连接 WSL

vscode中 ctrl+shift+p 选择wsl 在wsl中重新打开文件
命令
- gcc helloWorld.c -o ./hello
- ./hello