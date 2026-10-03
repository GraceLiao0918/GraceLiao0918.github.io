---
title: 🔖TodoLists
cssclasses:
  - hide-properties
banner: "![[TodoListImg.png]]"
banner_y: 0.2
---


## 📅日记传送
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
## 📌待办事项

- [ ] 深度学习的书籍[[2026-09-12]] [鱼书pdf - 知乎](https://zhuanlan.zhihu.com/p/665578698)
- [ ] Multimodal Controlled 3D Facial Animation and Conversation of Digital Human Teachers with Automatic Style Planning and Lightweight ResNet相关论文（Agent）[[2026-09-22]]
- [ ] Grasp for MOCDSP算法设计（Agent）[[2026-09-28]]


```tasks
not done
```


## ⏳历史任务
```tasks
done
```