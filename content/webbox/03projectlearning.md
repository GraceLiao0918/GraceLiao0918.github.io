---
title: 🪴 碎片化的前端知识
tags:
  - web
---

# 😼框架

## 字体

```
@font-face {
  font-family: "AlimamaDaoLiTi";
  src: url("@/assets/fonts/AlimamaDaoLiTi.ttf") format("truetype");
  font-weight: normal;
  font-style: normal;
}
body {
  font-family: AlimamaDaoLiTi;
}
```

## 路由

- 安装vue-router

```
npm install vue-router
```

- 配置路由文件

> src/router/index.js

```
// 项目的路由配置文件

import { createRouter, createWebHashHistory } from "vue-router";

import TablePage from "@/view/tablePage.vue";

const routes = [
  { path: "/", component: TablePage },
  { path: "/table", component: TablePage },
];

const router = createRouter({
  history: createWebHashHistory(),
  routes,
});

export default router;
```

> [!note] note
> 注意route和router的拼写以及区别
> 注册路由

> src/main.js

```
import router from "./router";

createApp(App).use(router).mount("#app");

```

- 修改App.vue使用<router-view>

```
<router-view></router-view>

```

# 😺模块

## 登录

- 表单验证
