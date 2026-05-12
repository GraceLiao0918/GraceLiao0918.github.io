---
title: 🫣开始Obsidian之旅
tags:
  - tool
---
## Obsidian概述

Obsidian 是一款优秀的笔记软件，是基于 markdown 语法的编辑器。它可以帮助你轻松地创建、整理和回顾自己的笔记。

## Obsidian插件

Calendar
Shiki Highlighter
Iconize
Tasks

## Obsidian个性化HomePage

### banner header 图片

> banners插件：[Release 1.3.3 · noatpad/obsidian-banners · GitHub](https://github.com/noatpad/obsidian-banners/releases/tag/1.3.3)

- 命令面板搜索并选择 ==Add/Change banner with local image== 命令选择本地图片作为笔记的头图
- 这个插件使用 YAML 语法，在笔记的 Frontmatter/元数据 区域来存储关于头图的信息。
```
#头图图片的源路径可以是URL或图片的内部链接。
#注意：请确保它用引号括起来，以避免分析错误，例如“![[file]]”
banner: string
# 头图的中心位置，number 在0-1之间的整数
banner_x: number
banner_y: number
# 头图是否锁定位置，不允许拖动更改聚焦的位置，比如你是一张海报，想定位到具体位置
banner_lock: boolean
# 横幅图标。string 可以是表情符号或任何字符串
banner_icon: string
```

### button 按钮

> buttons插件
- 使用
````
```button
name PKMer
type command
action 在新面板中打开光标处链接
```
# buttons id
^button-pkmer
````

| 按钮类型      | Button Type            | 介绍                                    |
| --------- | ---------------------- | ------------------------------------- |
| 命令        | command                | 选择要运行的命令面板命令                          |
| 链接        | Link                   | 写入 URL 或 URI                          |
| 计算        | calculate              | 编写数学公式                                |
| 模板 - 前置   | Prepend Template       | 点按按钮以将模板预置到当前笔记                       |
| 模板 - 追加   | Append Template        | 点按按钮以将模板追加到当前笔记                       |
| 模板 - 在行添加 | Add Template at Line   | 单击按钮将模板添加到指定行的当前注释中                   |
| 模板 - 新建笔记 | New Note From Template | 选取模板，输入新笔记的名称，选取是否应在拆分窗格中打开新笔记        |
| 文本 - 前置   | Prepend                | 点按按钮以在当前笔记前追加文本                       |
| 文本 - 追加   | Append                 | 点按按钮以将当前笔记末追加文本                       |
| 文本 - 在行添加 | Add Text at Line       | 单击按钮以将文本添加到指定行的笔记中                    |
| 文本 - 新建笔记 | New Note               | 写下新笔记的名称，选择是否应在新的面板中打开笔记              |
| 交换        | Swap                   | 写下每次点击时交换按钮的按钮块 ID，例如 [id1, id2] 按空格分 |

### dataview 插件

> 统计本文本库的文件、标签数量

```
```dataviewjs
let ftMd = dv.pages("").file.sort(t => t.cday)[0]
let total = parseInt([new Date() - ftMd.ctime] / (60*60*24*1000))
let totalDays = " 您已使用 *Obsidian* "+total+" 天，"
let nofold = '!"misc/templates"'
let allFile = dv.pages(nofold).file
let totalMd = "共创建 "+
	allFile.length+" 篇笔记"
let totalTag = allFile.etags.distinct().length+" 个标签"

dv.paragraph(
	totalDays+totalMd+"、"+totalTag+""
)
```

### 自定义CSS

- 打开笔记仓库的 `.obsidian` 文件夹，创建 `snippets` 文件夹。
- 代码片段（CSS）文件，放进 `snippets` 文件夹
- Obsidian > 设置 > 外观，最后一项【CSS 代码片段】，刷新一下，会显示出新增的文件，把后面的切换按钮打开即可。
### Task

- 设置 → Tasks → Global Filter
- 末尾粘贴：`hide toolbar`
- 重启 Obsidian，全库所有 Tasks 块都不再显示筛选栏

