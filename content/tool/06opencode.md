---
title: 🧆 opencode+skills
tags:
  - tool
---

## opencode概述

opencode是一个开源的 AI 编码代理。

### 代理

代理是专门的 AI 助手，可以针对特定任务和工作流程进行配置。它们允许您创建具有自定义提示词、模型和工具访问权限的专用工具。

OpenCode 内置了两个主代理：Build 和 Plan。

- 使用 Build
  - 模式：primary
  - Build 是启用了所有工具的默认主代理。这是用于需要完全访问文件操作和系统命令的开发工作的标准代理。

- 使用 Plan
  - 模式：primary
  - 一个专为规划和分析设计的受限代理。我们使用权限系统来为您提供更多控制权，并防止意外更改。 默认情况下，以下所有项均设置为 ask：
  - file edits：所有写入、补丁和编辑
  - bash：所有 bash 命令
  - 当您希望 LLM 分析代码、建议更改或创建计划，而不对代码库进行任何实际修改时，此代理非常有用。

OpenCode 内置了两个子代理：General 和 Explore。

- 使用 General
  - 模式：subagent
  - 一个用于研究复杂问题和执行多步骤任务的通用代理。拥有完整的工具访问权限（todo 除外），因此可以在需要时修改文件。可用于并行运行多个工作单元。

- 使用 Explore
  - 模式：subagent
  - 一个用于探索代码库的快速只读代理。无法修改文件。当您需要按模式快速查找文件、搜索代码中的关键字或回答有关代码库的问题时，请使用此代理。

### 模型

配置 LLM 提供商和模型。

### 安装

- vscode插件市场安装opencode插件
- 全局安装opencode cli `npm install -g opencode-ai`
- opencode地址 `C:\Users\liao'zi'qing\.config\opencode`

## skills概述

> https://zhuanlan.zhihu.com/p/1999979760458167377

### skills是什么

Skills 本质上是一组指令、脚本和资源的集合，AI 会在需要时动态加载它们。你可以把它理解成 AI 的"专业技能证书"——有了对应的 Skill，AI 就知道该怎么按照特定的方式完成任务。

每个 Skill 都是独立的文件夹，里面包含一个 `SKILL.md` 文件，这个文件定义了 AI 需要遵循的指令和元数据。

## opencode+skills

- 目录结构

```
.opencode/
└── skills/
    ├── xiaohongshu-creator/<-- 技能1：小红书
        └── SKILL.md
    ├── svg-animation-expert/<-- 技能2：SVG动画
        └── SKILL.md
    └── modern-ui-expert/<-- 技能3：前端UI
        └── SKILL.md

```

- 配置opencode.json文件

```
{
    "$schema": "https://opencode.ai/config.json",
    "permission": {
        "skill": {
            "*": "allow",
            "pr-review": "allow",
            "internal-*": "deny",
            "experimental-*": "ask"
        }
    }
}

```

- 核心文件 SKILL.md

```
---
name: my-skill-name<-- 必须是小写、数字、单连字符 (正则: ^[a-z0-9]+(-[a-z0-9]+)*$)
description: 一句话描述这个技能是干嘛的，1024字符以内。
license: MIT
compatibility: opencode
metadata:
version:1.0.0
---
## What I do
这里写具体的技能描述...
## When to use me
这里告诉 AI 什么时候触发这个技能...

```

- 使用

在 OpenCode中，只要文件存在，你在对话框输入技能名或直接提问，AI 就会自动加载。
