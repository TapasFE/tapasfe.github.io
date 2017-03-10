---
title: 如何在本博客发布文章
date: 2017-03-10 14:21:10
tags:
---

### 初始化
1. 从[TapasFE代码仓库](https://github.com/TapasFE/tapasfe.github.io)克隆到本地
2. 切换分支到 posts
```bash
git checkout posts
```
3. 安装dependencies
```bash
npm i
```

### 本地开发
1. 全局安装Hexo命令行工具
```bash
npm i hexo-cli -g
```
2. 启动本地服务器
```bash
npm run dev
```

  此时，可在localhost:4000查看博客网站

### 编写新的博客
1. 创建博客
```bash
npm run new 'my-blog-title'
```
  Hexo会自动在`/source/_posts/`路径下创建名为'my-blog-title.md'的文件，并自动根据发布日期创建`/yyyy/mm/dd/my-blog-title/`的路由。
2. 编写博客
  编辑`/source/_posts/my-blog-title.md`文件，同时browserSync插件会自动把内容更新到浏览器上，便于即时查看修改。

### 发布博客
1. 打包静态资源
```bash
npm run build
```
  静态资源打包后放在`/public`路径下
2. 发布上线
```bash
npm run release
```
  部署`/public`路径下的所有文件到远程
  
  ***另：1、2两步也可以合并为以下命令，依次完成打包和上线***
```bash
npm run publish
```

### 删除已发布的博客
1. 删除本地Markdown文件
2. 清除打包过的带有待删除博客的静态文件
```bash
npm run clean
```
3. 重新打包上线
```bash
npm run publish
```