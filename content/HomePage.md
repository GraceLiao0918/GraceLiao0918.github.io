---
banner: "![[background01.png]]"
banner_y: 0.28
banner_string: Home
cssclasses:
  - colse_show_title
  - full_width_page
  - hide-properties
---

---


# (✿◕‿◕✿)Welcome to llotso!💕


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

```dataviewjs
const allPages = dv.pages();
const tagCounts = {};

allPages.forEach(page => {
    if (page.file && page.file.tags) {
        const tags = page.file.tags;
        tags.forEach(tag => {
            tagCounts[tag] = (tagCounts[tag] || 0) + 1;
        });
    }
});

const sortedTags = Object.entries(tagCounts)
    .sort((a, b) => b[1] - a[1])
    .slice(0, 16);

const getColor = (index) => {
    const colors = ['#667eea', '#f093fb', '#4facfe', '#00f2fe', '#11998e', '#38ef7d', '#fa709a', '#fee140'];
    return colors[index % colors.length];
};

const tagHtml = sortedTags.map(([tag, count], index) => 
    `<span style="display: inline-block; background: ${getColor(index)}22; border: 0px solid ${getColor(index)}; color: ${getColor(index)}; padding: 4px 10px; border-radius: 12px; margin: 3px; font-size: 22px;">${tag} (${count})</span>`
).join('');

dv.paragraph(`<div style="display: flex; flex-wrap: wrap; gap: 4px;">${tagHtml}</div>`);
```

```dataviewjs
const pages = dv.pages('""');
const allTasks = pages.file.tasks;

const total = allTasks.length;
const completed = allTasks.where(t => t.completed).length;
const incomplete = total - completed;

const today = dv.date('today');
const startOfWeek = today.startOf('week');
const monday = startOfWeek.plus({ days: startOfWeek.weekday === 7 ? 1 : 0 });
const sunday = monday.plus({ days: 6 });
const completedThisWeek = allTasks
    .where(t => t.completed && t.completion && t.completion >= monday && t.completion <= sunday)
    .length;

const progress = total > 0 ? Math.round((completed / total) * 100) : 0;

dv.paragraph(`
<div style="margin: 10px 0;">
    <div style="display: grid; grid-template-columns: repeat(3, 1fr); gap: 8px; text-align: center;">
        <div style="padding: 12px 6px; background: #fff3cd; border-radius: 10px; border: 0px solid #ffc107;">
            <div style="font-size: 22px; font-weight: bold; color: #856404;">${incomplete}</div>
            <div style="font-size: 22px; color: #856404;">待完成</div>
        </div>
        <div style="padding: 12px 6px; background: #d4edda; border-radius: 10px; border: 0px solid #28a745;">
            <div style="font-size: 22px; font-weight: bold; color: #155724;">${completed}</div>
            <div style="font-size: 22px; color: #155724;">已完成</div>
        </div>
        <div style="padding: 12px 6px; background: linear-gradient(135deg, #11998e, #38ef7d); border-radius: 10px; color: white;">
            <div style="font-size: 22px; font-weight: bold;">${progress}%</div>
            <div style="font-size: 22px; opacity: 0.9;">完成率</div>
        </div>
    </div>
    <div style="margin-top: 10px; padding: 10px; background: linear-gradient(135deg, #f093fb, #f5576c); border-radius: 8px; text-align: center; color: white; font-size: 13px;">
        📍 本周已完成 <b>${completedThisWeek}</b> 个任务
    </div>
</div>
`);
```

```dataviewjs
const now = new Date();
const today = now.toISOString().split('T')[0];

const yesterdayDate = new Date(now);
yesterdayDate.setDate(yesterdayDate.getDate() - 1);
const yesterday = yesterdayDate.toISOString().split('T')[0];

const weekAgoDate = new Date(now);
weekAgoDate.setDate(weekAgoDate.getDate() - 7);
const weekAgo = weekAgoDate.toISOString().split('T')[0];

const monthAgoDate = new Date(now);
monthAgoDate.setDate(monthAgoDate.getDate() - 30);
const monthAgo = monthAgoDate.toISOString().split('T')[0];

const allPages = dv.pages().array();
let todayCount = 0;
let yesterdayCount = 0;
let weekCount = 0;
let monthCount = 0;

allPages.forEach(page => {
    if (page.file && page.file.mtime) {
        const pageDate = page.file.mtime.toISODate();
        if (pageDate === today) todayCount++;
        if (pageDate === yesterday) yesterdayCount++;
        if (pageDate >= weekAgo) weekCount++;
        if (pageDate >= monthAgo) monthCount++;
    }
});

const totalCount = allPages.length;

dv.paragraph(`
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; margin: 10px 0;">
    <div style="background: linear-gradient(135deg, #11998e, #38ef7d); padding: 15px; border-radius: 14px; text-align: center; color: white; box-shadow: 0 6px 15px rgba(17, 153, 142, 0.25);">
        <div style="font-size: 28px; font-weight: 800;">${todayCount}</div>
        <div style="font-size: 22px; opacity: 0.95;">🌅 今日编辑</div>
    </div>
    <div style="background: linear-gradient(135deg, #4facfe, #00f2fe); padding: 15px; border-radius: 14px; text-align: center; color: white; box-shadow: 0 6px 15px rgba(79, 172, 254, 0.25);">
        <div style="font-size: 28px; font-weight: 800;">${weekCount}</div>
        <div style="font-size: 22px; opacity: 0.95;">📅 本周活跃</div>
    </div>
    <div style="background: linear-gradient(135deg, #f093fb, #f5576c); padding: 15px; border-radius: 14px; text-align: center; color: white; box-shadow: 0 6px 15px rgba(245, 87, 108, 0.25);">
        <div style="font-size: 28px; font-weight: 800;">${monthCount}</div>
        <div style="font-size: 22px; opacity: 0.95;">🗓️ 本月贡献</div>
    </div>
    <div style="background: linear-gradient(135deg, #667eea, #764ba2); padding: 15px; border-radius: 14px; text-align: center; color: white; box-shadow: 0 6px 15px rgba(102, 126, 234, 0.25);">
        <div style="font-size: 28px; font-weight: 800;">${totalCount}</div>
        <div style="font-size: 22px; opacity: 0.95;">📚 知识库总量</div>
    </div>
</div>
`);
```

```dataviewjs
const today = dv.date('today');
const yesterday = today.minus(dv.duration('1 day'));
const tomorrow = today.plus(dv.duration('1 day'));
const formatDate = (date) => date.toFormat('yyyy-MM-dd');

dv.list([
    `[[${formatDate(yesterday)}|◀ 昨天]]`,
    `**[[${formatDate(today)}|📅 今天]]**`,
    `[[${formatDate(tomorrow)}|明天 ▶]]`
]);
```



- 团子杂货铺
    - 标签： #tool 
    - 简介：藏满小妙招的便利小仓库
- 算法小铺
    - 标签： #algorithm 
    - 简介：装满解题思路的灵感货架
- 课堂小铺
    - 标签： #class 
    - 简介：整理课程知识的学习小卖部
- 机器学习小铺
    - 标签： #machineLearning 
    - 简介：让模型慢慢变聪明的训练货架
- 强化学习小铺
    - 标签： #ReinforcementLearning 
    - 简介：在奖励中越练越强的闯关小店
- 模板小铺
    - 标签： #templates 
    - 简介：拿来就能用的版式仓库
- 幻灯片小铺
    - 标签： #slides 
    - 简介：装着演示页面的放映杂货铺
- 日记小铺
    - 标签： #diary 
    - 简介：记录日常碎碎念的温柔储藏柜

| 文件夹                   | 名称     | 标签                     | 简介             |
| --------------------- | ------ | ---------------------- | -------------- |
| algorithm             | 算法小铺   | #algorithm             | 装满解题思路的灵感货架。   |
| assets                | 素材小铺   | #assets                | 收藏各种可爱资源的百宝阁。  |
| class                 | 课堂小铺   | #class                 | 整理课程知识的学习小卖部。  |
| diary                 | 日记小铺   | #diary                 | 记录日常碎碎念的温柔储藏柜。 |
| images                | 图册小铺   | #images                | 存放美好画面的相片杂货店。  |
| MachineLearning       | 机器学习小铺 | #MachineLearning       | 让模型慢慢变聪明的训练货架。 |
| PrivateTutor          | 私教小铺   | #PrivateTutor          | 一对一知识补给站。      |
| ReinforcementLearning | 强化学习小铺 | #ReinforcementLearning | 在奖励中越练越强的闯关小店。 |
| slides                | 幻灯片小铺  | #slides                | 装着演示页面的放映杂货铺。  |
| templates             | 模板小铺   | #templates             | 拿来就能用的版式仓库。    |
| tool                  | 工具小铺   | #tool                  | 藏满小妙招的便利小仓库。   |
| webbox                | 网页小铺   | #webbox                | 收纳网页片段的小站点。    |
| about                 | 关于小铺   | #about                 | 介绍自己和本站的小角落。   |
| HomePage              | 首页小铺   | #HomePage              | 欢迎来到知识杂货铺。     |
| index                 | 目录小铺   | #index                 | 快速找到内容的索引货架。   |
| TodoLists             | 待办小铺   | #TodoLists             | 把计划一一收好的清单小店。  |
