---
title: Quartz博客框架使用方法
tags:
  - tool
---

<!-- # Quartz博客框架使用方法 -->

## 😀 快速上手

- 初始化Quartz

> Quartz需要[Node](https://nodejs.org/)版本v18.14和`npm`版本v9.3.1以上才能正常工作。

```powershell
git clone https://github.com/jackyzha0/quartz.git
cd quartz
npm i
npx quartz create
```

- 本地浏览器打开

```powershell
npx quartz build --serve
```

## 😎 Quartz基础配置

基础配置需要在 `quartz.config.ts` 中进行配置，具体参数详情见[官方文档](https://quartz.songxingguo.com/configuration)

- 日期的修改
  - 设置为中文：`locale: "zh-CN"`
  - 日期显示:`defaultDateType: "modified"`

- 字体的修改
  - 在[阿里巴巴矢量图标库](https://www.iconfont.cn/)中下载字体包

  - 在`public/static`下新建`fonts`文件夹，添加字体包中的文件

  - 在`quartz/style`下`custom.css`中引入字体

    ```css
    @font-face {
      font-family: "阿里妈妈刀隶体 Regular";
      font-weight: 400;
      src:
        url("../static/fonts/AlimamaDaoLiTi.woff2") format("woff2"),
        url("../static/fonts/AlimamaDaoLiTi.woff") format("woff");
      font-variation-settings: normal;
      font-display: swap;
    }
    ```

  - 在`quartz.config.ts`使用字体

    ```typescript
    fontOrigin: "local",
    cdnCaching: false,
    typography: {
      header: "阿里妈妈刀隶体 Regular",
      body: "阿里妈妈刀隶体 Regular",
      code: "阿里妈妈刀隶体 Regular",
    }
    ```

- 主题样式的修改

  ```typescript
  theme: {
    fontOrigin: "local",
    cdnCaching: false,
    typography: {
      header: "阿里妈妈刀隶体 Regular",
      body: "阿里妈妈刀隶体 Regular",
      code: "阿里妈妈刀隶体 Regular",
    },
    colors: {
      lightMode: {
        light: "#faf8f8",
        lightgray: "#e5e5e5",
        gray: "#b8b8b8",
        darkgray: "#444f5a",
        dark: "#3e4149",
        secondary: "#ff9999",
        tertiary: "#ffc8c8",
        highlight: "rgb(255, 226, 226,0.45)",
        textHighlight: "#fff23688",
      },
      darkMode: {
        light: "#161618",
        lightgray: "#393639",
        gray: "#646464",
        darkgray: "#d4d4d4",
        dark: "#ebebec",
        secondary: "#7b97aa",
        tertiary: "#84a59d",
        highlight: "rgba(143, 159, 169, 0.15)",
        textHighlight: "#b3aa0288",
      },
    },
  }
  ```

## 😁 编写内容

Quartz中的所有内容都应放在`/content`文件夹中。Quartz主页的内容位于`content/index.md`中。

```markdown
---
title: Example Title
draft: false
tags:
  - example-tag
---

The rest of your content lives here. You can use **Markdown** here :)
```

- `title`: 页面的标题。如果没有提供，Quartz将使用文件名作为标题。
- `description`: 用于链接预览的说明。
- `aliases`: 笔记的其他名称。为字符串列表。
- `tags`: 笔记的标签。
- `draft`: 是否发布页面。这是在Quartz中创建private pages的一种方法。
- `date`: 表示发布日期的字符串。通常使用“YYYY-MM-DD”格式

## 😗 主页盒子组件

- 在`quartz/components`下新建`Landing.tsx`,编写组件内容
- 图片等资源在`quartz/status`中

- 在`quartz/components/styles`下新建`Landing.scss`,编写组件的样式
- 在`quartz/components`下`index.tsx`中引入组件
- 在`quartz.layout.ts`使用组件

```typescript
Component.ConditionalRender({
  component: Component.Landing(),
  condition: (page) => page.fileData.slug == "index",
}),
```
