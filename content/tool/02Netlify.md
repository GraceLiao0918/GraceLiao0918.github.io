---
title: Netlify 部署应用
tags:
  - tool
---

- 访问Netlify官网并注册一个免费账户。可以选择通过GitHub、GitLab或Bitbucket进行注册。在这里我选择GitHub注册。

- 准备网站源码，将项目源码提交到GitHub。

- 登录Netlify后，点击“New site from Git”，选择代码托管平（GitHub），并授权Netlify访问你的仓库。

- 在“Deploy settings”页面，选择希望Netlify监听的分支（通常是main或master）。在“Build command”字段中，输入项目所需的构建命令（如*npm run build*），在“Publish directory”字段中，输入构建完成后生成的静态文件所在的目录（如*build*或*dist*）。

- 完成上述配置后，点击“Deploy site”按钮，Netlify将开始自动构建并部署网站。部署完成后，Netlify会为你生成一个默认的子域名（如*your-site.netlify.app*），可以通过该域名立即访问网站。
