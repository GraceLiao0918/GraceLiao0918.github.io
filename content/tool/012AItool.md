---
title: 🪇 AItool
tags:
  - AItool
  - tool
---

# Agent = LLM + Harness

## What is Harness

> [!note]
>
> Harness 是让 LLM 获取上下文、调用工具并接收执行结果的运行环境。[Claude Code 工作机制](https://code.claude.com/docs/en/how-claude-code-works)
## So many LLMs and Harnesses

> [!note]
>
> - 模型 $\mathcal{M}=\{\text{GPT},\text{Claude},\text{Gemini},\text{GLM},\text{Kimi},\text{DeepSeek}, \dots\}$。
> - Harness $\mathcal{H}=\{\text{Codex CLI},\text{Claude Code},\text{OpenCode},\text{Pi},\text{DSH},\dots\}$（[Codex CLI](https://developers.openai.com/codex/cli/) · [Claude Code](https://code.claude.com/docs/en/overview) · [Pi](https://pi.dev/)）。
## What I use: Model

- 主力模型：SOTA 的模型，包括 GPT-6-Astra, Fable-5.1
- 快速模型：DeepSeek-Flash-V4.1, Gemini-Flash-3.8

> [!note]
> 可以看**价格-能力图**来选择合适的模型
### Main Model

主力模型一般用来做些复杂、困难的事情：

> [!note]
>
> - 找研究 idea：先提出问题，再检查是否值得做。
> - 讨论方案，把最终决定写成设计文档。
> - 有 Sub-Agent 时，分配任务并核对结果。


**我的主力模型：**

- **GPT-6-Astra**: 最常用的模型，GPT 的订阅比较容易（可以考虑通过 Google Pay 的方式在日区进行订阅，要求一张外币支付卡，万事达/VISA），且 GPT 经常重置额度（[Tibo 主页](https://x.com/thsottiaux?lang=en)），花费通常来说比较值得

- **Fable-5.1**: 能力较强，但由于 `Anthropic` 封号严重，一般来说只能通过中转站，花费太高且很容易被路由到其他模型，没有订阅的情况下不是很推荐

**论坛评价：**

> [!note]
>
> - GPT-6-Astra —— 推理与科学基准、长链路任务（[X·OpenAI](https://x.com/OpenAI/status/2095595752815030713) · [知乎](https://www.zhihu.com/question/2079625515565060391) · [HN](https://news.ycombinator.com/item?id=49714978)）
> - Fable-5.1 —— 长程科研与知识工作（[X·Anthropic](https://x.com/bcherny/status/2094864060609376748) · [知乎](https://www.zhihu.com/question/2078371741722850186) · [HN](https://news.ycombinator.com/item?id=49525617)）
## Flash Model

这种模型一般用来：

1. 作为 sub-agent 使用，但只是为了获得更快的响应和更少的钱，不保证性能不损失
2. 一些简单事情的问答

> [!note]
>
> - 我比较常用的是 DeepSeek-Flash-V4.1 来做些简单的事情：翻译，解释代码等等
> - 问答常用 Gemini，Gemini 的世界知识比较好，One-Shot 能力很强，但多轮对话会原形毕露（仅限 Web）
## What I Use: Harness

我使用的 Harness 与 Model 组合为：

- 官方组合：`Codex + GPT` + `Claude Code + Claude`
- 客制化组合：
  - `Pi + GPT/DeepSeek`
  - `Claude Code + DeepSeek`
### Harness

也可以考虑更方便的桌面版，使用简单，但可定制性更低

> [!note]
>
> - 官方：`ChatGPT Desktop(Codex) + GPT`（[文档](https://developers.openai.com/codex/app)）、`ZCode + GLM`（[官网](https://zcode.z.ai/)）、`DeepSeek Harness Desktop + DeepSeek`（[Windows](https://download.deepseek.com/dsh-desk/bin/win-x64/deepseek-harness-0.1.7-rc.2-win-x64.exe) / [macOS](https://download.deepseek.com/dsh-desk/bin/mac-arm64/deepseek-harness-0.1.7-rc.2-mac-arm64.dmg)，源码在 [apps/desktop](https://github.com/deepseek-ai/deepseek-harness/tree/main/apps/desktop)）。
> - ZCode 争议：被曝静默上传含完整 `.git` 历史的工作区快照到阿里云 OSS（[报道](https://www.sohu.com/a/1077913364_115060)）
### How to Use

我们可以通过 Harness 和 Skill 来最大限度的激活模型的能力

> [!note]
>
> - 项目约定写进 [`AGENTS.md`](https://agents.md)：模型每次都能读到，不必重复交代。
> - 固定流程做成 [Skill](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)：名字与描述常驻，正文按需加载。
> - 按任务挑组合和思考等级：官方组合求稳，客制化组合（`Pi + GPT`、`Claude Code + DeepSeek`）控成本。
> - 可验证的子任务交给快速模型并行做，冲突或反复失败时交回主力模型。
## Skills

本质上是提示词 + 流程编排

> [!node]
>
> - 把反复用到的做法固化成[文件](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)：名字与描述常驻上下文，正文按需加载。
> - 适合跨会话的固定工作流：同一套步骤与约定只写一次，之后按需触发。

**什么是有用的 Skill？**

- 通用的 Skill 都没有什么用，模型的后训练会将这些 Skill 都融入到模型能力中
- 编排流程过细的 Skill 没有什么用，[会影响模型的能力](https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra)

> [!important]
> 目前来看，项目级别的 Skill 是最有用的

### A Session is a Skill

如何写出这样的 Skill？

一个 `Session` 就是一个 `Skill`：

> [!note]
>
> - 先跑一遍：在一次对话里把事做通，把过程抽象成一个 Loop（输入 → 步骤 → 验收标准）。
> - 再固化：把这个 Loop 交给 [`@skill-creator`](https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra) 生成 Skill，不必从零手写，产物天然是项目级别。

例如：
- [`@init-slides-os`](https://github.com/virgiling/Slides-Template/blob/main/.agents/skills/init-slides-os/SKILL.md)

> [!note]
>
> 把新克隆的模板初始化成可用的 slides 工作区：检查依赖、复制公开资源，并生成外层 `make-marp-slides` 入口。

- [`@commit-deploy`](./assets/commit-deploy-skill.md)

> [!note]
>
> 把一次发布做成闭环：限定范围提交 → 发布一个指定目标 → 核验该版本已上线。
### A Bitter Lesson From Software Engineer

在 Agentic RL 时代，我们都变成了产品经理，[软件工程中的很多理论我们都可以拿来使用](https://jyywiki.cn/GSE/2026/)

这里最想说的就是[《人月神话》](https://cactus-proj.github.io/The-Mythical-Man-Month-zh/)中的关于偶然复杂度和必然复杂度的概念：

> [!aigc]
>
> - 必然复杂度：问题本身有多难——要做什么、概念结构如何构思，换工具也消不掉。
> - 偶然复杂度：实现过程有多难——语言、工具、环境的摩擦，更好的工具能消除。
>
> 出处：[第 16 章 · 没有银弹（根本和次要问题）](https://cactus-proj.github.io/The-Mythical-Man-Month-zh/ch16.html)、[第 17 章 · 再论《没有银弹》](https://cactus-proj.github.io/The-Mythical-Man-Month-zh/ch17.html)

## Main Loop

> 软件是 Intent、Spec、Impl 的组合体

一个非常重要的观点就是：

> [!note]
>
> - 必然复杂度只能靠人：动手前先理清需求与系统边界。
> - 但不必一次想全：只钉死已决定、违反代价高的骨架，其余显式留白（「有意的欠规约」）。
> - 偶然复杂度可以交给 Agent：用 `TDD` (Test-driven development) 自我验证、自我提升。

所以我们的流程可以简化为：

> [!note]
>
> 1. **Spec**：意图 → 需求、边界、验收标准。
> 2. **Impl**：Agent 动手，测试先行。
> 3. **Verify**：跑测试与不变量，核对已决定项。
> 4. **Loop**：先做 MVP（Minimum Viable Product）让假设尽早遇到证据；需求变了就回 1，版本化留档。

总结就是 `Code is cheap, show me the document`

## What if the documents were not readable?

当文档：太长/太难/术语太多/流程不够明显全是文字 不想看的时候，我们可以考虑使用这类 `Skill`：

- [`@HumanLayer/show-me`](https://github.com/humanlayer/skills/blob/main/plugins/show-me/skills/show-me/SKILL.md)
- [`@Nico Bailon/visual-explainer`](https://github.com/nicobailon/visual-explainer)

这样我们就可以得到一个比较好看的模板，比如我们可以可视化[这个 Slides 模板系统的文档](./assets/template-design.html)，或者可视化得到[我的个人Harness设计](./assets/personal-harness-overview.html)

## Evidence is Important

把关键产出留在文件里，让证据成为项目的 Memory，而不只留在对话里

> [!note]
>
> Mario Zechner：[“Prompts are code, .json/.md files are state”](https://mariozechner.at/posts/2025-06-02-prompts-are-code/)——将状态持久化到文件，用新上下文继续工作。
>
> - **接续任务**：进度文件记录已完成、待办与阻塞，新会话不必从头摸索。
> - **保留决策**：设计文档写清选择、理由与适用边界，避免反复推翻已有结论。
> - **复核实验**：结论关联代码版本、配置和原始结果，不只留一句“效果更好”。
> - **避免重犯**：失败记录保留复现步骤与验证结果，修复后补回归测试。

### What Does Evidence Look Like

我们可以考虑用 research 的手段：

- 例如[用写论文的方式说明自己的设计](https://x.com/roifex/status/2095051796372951538)，可以参考[这个例子](./assets/distributed-mcs-design.html)

> [!note]
>
> 在有限篇幅里，让非本方向的 reviewer 看懂：**问题是什么、为何这样设计、凭什么相信有效**。
> 用一张总览图讲结构，一张对比表讲取舍，再用实验支撑结论，而不是堆实现细节。

- 数据统计分析也可以用 research 的手段

> [!note]
>
> - **先问问题，再看数据**：确定假设、指标与对照，统一数据口径，避免只挑好看的结果。
> - **既看收益，也看波动**：重复实验，报告差异大小与不确定性；用消融检查收益来自哪里。
> - **把分析变成证据**：保留数据、配置和脚本，用结果决定下一步实验，也写清失败与适用边界。

> [!note]
> 这个做法需要你自己知道自己应该看什么数据，需要看什么数据，否则直接让AI去看是没有意义的
## When Should You Use Your Own Harness?

当你觉得：

> [!note]
>
> - **不够灵活**：工具和流程太固定，接不上自己的工作方式。
> - **长程不稳**：任务一长就忘约定、偏目标，现有机制难以纠正。
> - **记忆难用**：上下文、项目状态和证据如何保留，自己说了不算。
> - **难以排查**：只看到结果，看不清哪里出了问题，也改不了。

可以从 `Pi`、`DSH` 这类可扩展的 Harness 入手，用插件补齐自己的需求，不必从零重造。

### Tips While Using Harness

当使用 `Harness` 的时候，有几点需要注意，这里以 Codex + GPT 为例：

- 使用过程中切换**模型**和`Thinking Level`可能会导致 `KV-Cache` 失效（[OpenAI 自己的缓存机制](https://x.com/zjp1997720/status/2093148728907702675)）
- 不同难度的任务需要选择合适的 `Thinking Level`，但大多数难度的任务，最强模型的 `high` 都完全可以胜任，用过高的思考等级可能会导致[过度思考](https://claude.dev/blog/spending-your-effort/)
- 多用 [`grill-me`](https://github.com/mattpocock/skills/blob/main/skills/productivity/grilling/SKILL.md) 这样类似的技能，搞清楚自己的需求
- 科研中的一些规范落实成 Skill 其实非常好用，例如[消融实验](https://x.com/xuanwo/status/2095025829424333044)

> [!aigc]
>
> **先理清需求，按任务选思考等级，把有效流程沉淀为 Skill，最终用证据而不是思考时长验收。**

## Where to Learn?

- Twitter/X：很多营销/真有技术 的 post 会在这个上面发，比如 Jev 这种营销前几天特别火
- GitHub：一些比较知名/trending 比较高的仓库，可能会有新的东西
- 知乎：有些帖子和评论会比较有启发
- Linux do：薅羊毛聚集地，没什么技术
- 官方文档、更新日志、[开发者博客](https://mariozechner.at/)

但技术更新迭代太快：

- 有时候可能是 **学的越慢越好**
- 遇到想知道的、想学的，可以直接问 Agent，让它去做调研，用之前说的那些工作流就行

# Using Agent in a Balanced Way


> - idea 决定你的上限，但 Agent 帮不了你找 idea
> - 从文献里找idea，主要看文献做了什么，怎么做的，合不合理，缺点是什么，有什么方法更好解决缺点
> - Agent 负责把 idea 变成可验证的实现

## idea

idea 不是工作量，是概念层面的一跃。上限由它定，之后所有执行都只是把上限兑现多少。

> [!note]
> - 同一个实现能力，喂给不同 idea，产出天差地别。
> - 模型后训练已经吸收了绝大多数"通用技能"，**可复制的技巧在贬值，不可复制的判断在升值**。
> - Agent 能帮你写得更好、算得更快，但不能替你回答"这个问题值得做吗"。
 
判断一个 idea 值不值得做，可以用这个检查清单：

|   维度  |   问自己的问题   |
| --- | --- |
| 新颖性 | 它是已有工作的组合，还是真正空着的一块？ |
| 可行性 | 以我的资源（时间/数据/算力）能推到出结论吗？ |
| 价值 | 做成了谁在乎？解决的是真问题还是自嗨？ |
| 可证伪 | 我能不能设计一个实验，让它可能被推翻？ |

### idea 来源

> [!important]
> 不读文献的 idea 叫"我以为"，读了文献的 idea 才叫"判断"。
> Agent给你的idea普遍是现在已经有的方法和技术拼凑的，可能是不合理的，甚至是不正确的。

> [!note]
> - **找矿**：用学术检索（scholar_search、数据库）按方向、概念、热点扫一圈，先摸清地形，不急着下结论。
> - **读矿**：精读几篇关键论文，不只是摘要——搞清问题、方法机制、关键证据，才知道缝在哪里。
> - **炼矿**：把多篇的结论交叉对比，找争议、找空隙、找"没人补上的那一步"，那就是你的 idea。
> - **存矿**：把找到的 idea、来源、出处留档，成为项目的 Memory，不只在对话里。

**文献给你什么，Agent 给不了什么：**

| 来自文献 | 来自 Agent |
| --- | --- |
| 什么已被做过了（避坑） | 把想法跑成原型（提速） |
| 哪里还空着（定位 gap） | 检索、整理、交叉综合（省力） |
| 哪些假设站得住（定方向） | 代码实现、验证、返工（闭环） |

> [!note]
> 一句话：**Agent 帮你"读得更多、跑得更快"，文献帮你"想得对"。** 前者省时间，后者定上限。

## Use Agent

> [!important]
> - Agent在写代码时，往往会产生大量的参数，这是不太合理的，后期调参和消融实验都很复杂，需要慎重设置参数
> - Agent写的代码会将代码变得复杂，会产生很多不必要的参数和函数，可维护性差，规范性不好，需要修改
> - 可以参考ai作图的配色，但是别用ai作图，太假。
> - 消融实验的重要性，可以让Agent辅助
> - 扬长避短，把长处发挥好。


当 idea 已经想清楚，剩下的复杂度就该交给 Agent：

> [!note]
> 1. **Spec**：把 idea 写成需求、边界、验收标准——这是人必须钉死的骨架。
> 2. **Impl**：Agent 动手，**测试先行**（TDD），让假设尽早遇到证据。
> 3. **Verify**：跑测试与不变量，核对已决定项，抽样重算关键数据。
> 4. **Loop**：先做 MVP，让假设尽早被推翻；需求变了就回 1，版本化留档。

> [!important]
> 边界在哪：**先理清需求，按任务选 Agent，把有效流程沉淀成 Skill，用证据而不是"思考时长"来验收。** 别让 Agent 替你拍板 idea——那是你的上限，也是它的盲区。

## Reusable Loops


> [!note]
> - **只靠 Agent 找方向**：检索快、产出多，但没有 idea 的锚点，得到的是"全面却空洞"的堆料。典型症状：资料一大把，结论说不清。
> - **只靠自己闷头实现**：上限锁死在自己已知的范围，且把时间烧在 Agent 三分钟能跑完的重复劳动上。典型症状：很努力，没增量。


> [!important]
> 把"找 idea → 定边界 → 交给 Agent 实现 → 用证据验收"这条 loop 固化下来，就是最有价值的项目级 Skill。
> **idea 决定你的上限，文献喂给你 idea，Agent 负责把 idea 跑成证据——各归其位，才是平衡。**


# AI tool
## AI Agent
### Gemini

https://gemini.google.com/

> [!failure] failure
> something went wrong
> 地址后加/gems/create

### chatGPT



### github copilt

教育认证优惠，目前只能使用GitHub copilt的auto模型

认证教程参考：https://blog.csdn.net/PGJ_168/article/details/150267222


## AI Token

- [xhyapi](https://xhyapi.com/)
- [Kimi API 开放平台](https://platform.kimi.com/?from=kimi_homepage_sidebar)

## AI提示词

### 关闭会话前的好习惯

关掉长会话前，先让它写一份交接文档直接输入:这个会话要结束了，请写一份交接文档存到HANDOFFtmd:

>我们在做什么任务、已经完成了什么、当前卡在哪、下一步计划是什么、有哪些踩过的坑绝对不要踩。写给一个完全没有上下文的新对话看

下次开新会话

>第一句话只要说:先读HANDOFF文件

### 别让Codex只会听你的

我发现Codex有个问题:你说什么，它有时候就信什么。哪怕你的思路本身有问题，它也可能顺着你的方向继续写。所以我在自定义指令里，加了这段提示词:

1. 执行前，先检查我的思路有没有错误、漏洞或遗漏。
2. 不要一味迎合我，要独立判断。
3. 涉及数据、人物和结论，尽量核实来源。如果我说错了，直接指出，并告诉我为什么。
4. 主动提醒我忽略的变量、成本和风险。

会听会写代码的Al很多，展开但敢反驳你的AI，才是真的好用。


###

- Step 1：划定研究范围
做法：选定你感兴趣的AI子方向（比如：多模态大模型的对抗攻击、LLM的推理能力增强），找2-3篇近期的顶会文章（最好带代码）当锚点，丢给AI让它总结：

每篇解决了啥、怎么解的、还有啥坑。

让AI根据这些，帮你梳理出几个结构性空白（比如“现有防御都假设X，但如果X不成立呢？”）。

关键：让AI先遍览顶会论文，再谈创新，这是防止想法飘在空中或撞车的基础。

- Step 2：挖假设，不要问怎么改进
做法：不要直接问“怎么改进XXX”，让AI换个思路： “假如我们是审稿人，请列举这个方向最火的3个方法，它们为什么会失败？在什么场景下会崩？”

这样做的意义是从“怎么加模块”转向“现有方法为什么失效”，这种视角更容易挖出反直觉的、有根有据的创新假设。

- Step 3：让AI自己迭代找坑
做法：拿到初步idea后，开新窗口，让AI扮演“最刻薄的审稿人”+“最较真的合作者”：

指出所有逻辑硬伤和漏洞。

追问每个假设是否站得住脚。

推荐需要对比的基准和文献。

迭代2-3轮，把idea的薄弱点炸一遍。很多听起来make sense但经不起推敲的假想法，在这一步就会被筛掉。

- Step 4：多模型交叉验证，避免被AI带偏
做法：把打磨过的idea分别丢给GPT 、Claude、Kimi 等不同模型，用同样的问题让它们互评。AI有讨好倾向，单模型容易自嗨，多个模型交叉检查能暴露低级bug和幻觉引用。

- Step 5：回归学术搜索引擎
做法：AI给出的idea再漂亮，也要自己去Google Scholar搜一圈：

这个概念/术语，是不是已经有人用别的名字发过了？

最近3年相关论文数量趋势如何？是红海还是死水？

关键：以同行评审过的论文为准绳，不要轻信AI编的研究现状。自己读原文，判断真实创新性和落地难度。

几点注意
对话超过20轮就让AI做交接总结，压缩成一段，换窗口无缝继续。

写prompt要把你想表达的说清楚、具体点就行，啰嗦点没关系。

保持独立思考，AI的优点是发散和查漏，但筛选和判断只能靠你自己对领域的理解积累。
