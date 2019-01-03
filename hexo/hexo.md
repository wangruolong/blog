---
title: 使用hexo搭建个人博客
author: 大大白
date: 2019-01-03 12:00:00
categories:
- hexo
- 个人博客
tags: 
- hexo
- 个人博客
---

# 概述
Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。详细可以查看https://hexo.io/zh-cn/docs/
<!-- more -->

## 安装
安装 Hexo 相当简单。然而在安装前，您必须检查电脑中是否已安装下列应用程序：
- Node.js
- Git

> npm install -g hexo-cli

# 建站
安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。

> hexo init <folder>
> cd <folder>
> npm install

新建完成后，指定文件夹的目录如下：
```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes

```
## _config.yml
网站的 配置 信息，您可以在此配置大部分的参数。

## package.json
应用程序的信息。EJS, Stylus 和 Markdown renderer 已默认安装，您可以自由移除。
```json
{
  "name": "hexo-site",
  "version": "0.0.0",
  "private": true,
  "hexo": {
    "version": ""
  },
  "dependencies": {
    "hexo": "^3.0.0",
    "hexo-generator-archive": "^0.1.0",
    "hexo-generator-category": "^0.1.0",
    "hexo-generator-index": "^0.1.0",
    "hexo-generator-tag": "^0.1.0",
    "hexo-renderer-ejs": "^0.1.0",
    "hexo-renderer-stylus": "^0.2.0",
    "hexo-renderer-marked": "^0.2.4",
    "hexo-server": "^0.1.2"
  }
}
```

## scaffolds
模版文件夹。当您新建文章时，Hexo 会根据 scaffold 来建立文件。

Hexo的模板是指在新建的markdown文件中默认填充的内容。例如，如果您修改scaffold/post.md中的Front-matter内容，那么每次新建一篇文章时都会包含这个修改。（说实话我还不太清楚这个怎么用。）

## source
资源文件夹是存放用户资源的地方。除 _posts 文件夹之外，开头命名为 _ (下划线)的文件/文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 public 文件夹，而其他文件会被拷贝过去。

## themes
主题 文件夹。Hexo 会根据主题来生成静态页面。hexo提供了丰富的主题可以选择，也可以自己制作主题然后发布上去。https://hexo.io/themes/

# 配置
您可以在 _config.yml 中修改大部分的配置。

## Site
配置网站的相关信息

```
title: 大大白的个人博客
subtitle: 人生就像一场旅行，身体和灵魂总有一个要在路上！
description: 沉淀下来的东西，才是真正属于自己的财富！
keywords: 技术博客
author: 大大白
language: zh-CN
timezone: Asia/Shanghai
```

## theme
配置主题，可以访问https://hexo.io/themes/，看到满意的主题下载到本地的themes目录下，并配置然后在_config.yml里面配置theme指向对应的目录。

```
theme: hexo-theme-next
```

## Deployment
部署。可以把生成的静态页面部署到git仓库。注意deploy的下面有空格。
```
deploy:
  type: git 
  repository: git@github.com:wangruolong/wangruolong.github.io.git
  branch: master
  message: deploy my blog
```

更多信息请查看https://hexo.io/zh-cn/docs/configuration

# 命令
## init

> hexo init [folder]

新建一个网站。如果没有设置 folder ，Hexo 默认在目前的文件夹建立网站。

## new

> hexo new [layout] <title>

新建一篇文章。如果没有设置 layout 的话，默认使用 _config.yml 中的 default_layout 参数代替。如果标题包含空格的话，请使用引号括起来。

## generate

> hexo generate
该命令可以简写为
> hexo g

生成静态文件。

|选项|描述|
|-------------|---------------------|
|-d, --deploy | 文件生成后立即部署网站|
|-w, --watch  | 监视文件变动         |

## server

> hexo server

启动服务器。默认情况下，访问网址为： http://localhost:4000/。

|选项|描述|
|-------------|-----------------------------|
|-p, --port	  |重设端口                      |
|-s, --static |只使用静态文件                |
|-l, --log	  |启动日记记录，使用覆盖记录格式  |

## deploy

> hexo deploy
部署网站。

|参数 |描述|
|----|----|
|-g, --generate|部署之前预先生成静态文件|

该命令可以简写为：
> hexo d

更多命令请查看https://hexo.io/zh-cn/docs/commands