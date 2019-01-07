---
title: 使用hexo搭建个人博客
author: 大大白
date: 2016-01-03 12:00:00
categories:
- 个人博客
tags: 
- 个人博客
---

Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

<!-- more -->

# 安装环境
安装 Hexo 相当简单。然而在安装前，您必须检查电脑中是否已安装下列应用程序：
- Node.js
- Git

```
npm install -g hexo-cli
```

# 初始化hexo的运行环境
安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。
```
hexo init [folder]
cd [folder]
npm install
```
初始化完成后，指定文件夹的目录如下：
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

# 文件目录说明
## _config.yml
hexo的配置文件。可以在这里配置站点名称、站点路径、主题、git仓库部署地址等。基本上hexo需要配置东西都在这里了。
### 配置`Site`
可以配置网站的相关信息。
```
title: 大大白的个人博客
subtitle: 人生就像一场旅行，身体和灵魂总有一个要在路上！
description: 沉淀下来的东西，才是真正属于自己的财富！
keywords: 技术博客
author: 大大白
language: zh-CN
timezone: Asia/Shanghai
```
### 配置`Theme`
可以配置网站的主题。需要注意的是，配置的主题需要先下载到本地`themes`文件夹里面，然后再配置`_config.yml`中的`theme`
```
theme: hexo-theme-next
```
### 配置`Deployment`
配置部署的仓库地址。可以把生成的静态页面部署到git仓库。注意deploy的下面有`tab`。
```
deploy:
  type: git 
  repository: git@github.com:wangruolong/wangruolong.github.io.git
  branch: master
  message: deploy my blog
```
更多配置信息请查看https://hexo.io/zh-cn/docs/configuration

## package.json
hexo依赖的包。EJS, Stylus 和 Markdown renderer 已默认安装，您可以自由移除。
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
模版文件夹。当您新建文章时，Hexo 会根据 scaffold 来建立文件。Hexo的模板是指在新建的markdown文件中默认填充的内容。例如，如果您修改scaffold/post.md中的Front-matter内容，那么每次新建一篇文章时都会包含这个修改。（说实话我还不太清楚这个怎么用。）
## source
资源文件夹是存放用户资源的地方。除 _posts 文件夹之外，开头命名为 _ (下划线)的文件/文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 public 文件夹，而其他文件会被拷贝过去。_posts里面存放的是对应访问域名地址里面的跟路径`/`和归档路径`/archives`。而其他路径就是对应source底下的文件夹，比如你想访问http://your-site/demo，的demo路径，那么应该在public里面创建demo文件夹，系统会访问demo里面的index.md文件。
## themes
主题文件夹。访问https://hexo.io/themes/选择喜欢的主题，下载下来之后，把整个文件夹放在`themes`目录下。同时还需要在上文中说到的hexo运行环境中的`_config.yml`的配置文件里面把`Theme`配置成文件夹名称才会生效。Hexo会根据你配置的邪恶主题来生成静态页面。每个主题里面都有`_config.yml`，可以在这个文件里面配置主题的菜单、国际化等信息。注意区分两个`_config.yml`。

# 命令
## 新建一个网站
如果没有设置 folder ，Hexo 默认在目前的文件夹建立网站。
```
hexo init [folder]
```
## 新建一篇文章
如果没有设置 layout 的话，默认使用 _config.yml 中的 default_layout 参数代替。如果标题包含空格的话，请使用引号括起来。
```
hexo new [layout] <title>
```
## 生成静态文件
```
hexo generate
//该命令可以简写为
hexo g
```
|选项|描述|
|-------------|---------------------|
|-d, --deploy | 文件生成后立即部署网站|
|-w, --watch  | 监视文件变动         |

## 启动服务器
默认情况下，访问网址为： http://localhost:4000/
```
hexo server
```
|选项|描述|
|-------------|-----------------------------|
|-p, --port	  |重设端口                      |
|-s, --static |只使用静态文件                |
|-l, --log	  |启动日记记录，使用覆盖记录格式  |

## 部署网站
把public文件夹里面的静态页面部署到在hexo中的`_config.yml`配置的仓库地址。
```
hexo deploy
//该命令可以简写为
hexo d
```
|参数 |描述|
|----|----|
|-g, --generate|部署之前预先生成静态文件|

更多命令请查看https://hexo.io/zh-cn/docs/commands


# 在使用hexo遇到的几个问题。
1. 发布新的主题没有生效。这个分两种情况，一种是`_config.yml`中的`site`的`root`路径配错了导致资源文件找不到，建议直接配置`/`。另一种情况一般是是git合并策略的问题，因为我们不断的`hexo generate`生产了许多重复的文件，git会认为这些文件无需替换（这个估计是git执行diff算法时候的bug吧，具体不是很清楚，我猜测的），所以导致我们`hexo deploy`之后，`hexo deploy`本质上也是`git push`当`git`认为你这些文件没有变化之后自然不会给你提交上去。最简单的解决办法就是，先`hexo clean`删除掉原来生成的静态页面，然后再`hexo deploy`，就肯定没问题了。
2. 更多详细信息可以查阅官方文档https://hexo.io/zh-cn/docs/
