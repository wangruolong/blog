---
title: 前端基础技能
author: 大大白
date: 2017-11-03 12:00:00
categories:
- 前端基础
tags: 
- 前端基础
---

前端基础知识积累。

<!-- more -->
# Node.js

- `Node.js` 中，`__dirname` 总是指向被执行js文件的绝对路径。

# path
要使用`path`需要先`npm install path`，然后再在需要的文件里面引入`const path = require('path');`才可以使用。
两种用法：
1. 连接路径：path.join([path1][, path2][, ...])
2. 路径解析：path.resolve([from ...], to)

```javascript
var path = require('path');
path.join('/foo', 'bar', 'baz/asdf', 'quux', '..')
// 连接后
'/foo/bar/baz/asdf'

path.resolve('/foo/bar', './baz')
// 输出结果为
'/foo/bar/baz'
path.resolve('/foo/bar', '/tmp/file/')
// 输出结果为
'/tmp/file'
path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif')
// 当前的工作路径是 /home/itbilu/node，则输出结果为
'/home/itbilu/node/wwwroot/static_files/gif/image.gif'

const path = require('path');
let myPath = path.join(__dirname,'/img/so');
let myPath2 = path.join(__dirname,'./img/so');
let myPath3 = path.resolve(__dirname,'/img/so');
let myPath4 = path.resolve(__dirname,'./img/so');
console.log(__dirname); //D:\myProgram\test
console.log(myPath);    //D:\myProgram\test\img\so
console.log(myPath2);   //D:\myProgram\test\img\so
console.log(myPath3);   //D:\img\so
console.log(myPath4);   //D:\myProgram\test\img\so
```

# ES6
任何人都可以向标准委员会（又称 TC39 委员会）提案，要求修改语言标准。
一种新的语法从提案到变成正式标准，需要经历五个阶段。每个阶段的变动都需要由 TC39 委员会批准。
Stage 0 - Strawman（展示阶段）
Stage 1 - Proposal（征求意见阶段）
Stage 2 - Draft（草案阶段）
Stage 3 - Candidate（候选人阶段）
Stage 4 - Finished（定案阶段）
一个提案只要能进入 Stage 2，就差不多肯定会包括在以后的正式标准里面。ECMAScript 当前的所有提案，可以在 TC39 的官方网站Github.com/tc39/ecma262查看。

# babel
Babel 的配置文件是`.babelrc`，存放在项目的根目录下。使用 Babel 的第一步，就是配置这个文件。
该文件用来设置转码规则和插件，基本格式如下。

```json
{
  "presets": [],
  "plugins": []
}
```

- `presets`字段设定转码规则，官方提供以下的规则集，你可以根据需要安装。

```
# 最新转码规则
$ npm install --save-dev babel-preset-latest
# react 转码规则
$ npm install --save-dev babel-preset-react
# 不同阶段语法提案的转码规则（共有4个阶段），选装一个
$ npm install --save-dev babel-preset-stage-0
$ npm install --save-dev babel-preset-stage-1
$ npm install --save-dev babel-preset-stage-2
$ npm install --save-dev babel-preset-stage-3
```

然后，将这些规则加入.babelrc。

```json
  {
    "presets": [
      "latest",
      "react",
      "stage-2"
    ],
    "plugins": []
  }
```














- React中对XSS如何进行XSS攻击和防范
- 依赖倒置，依赖抽象而不是具体实现
- 控制反转，依赖注入和依赖查找
- setTimeout()，newPromise()
- 浏览器事件循环，队列优先级不同
- 串行执行Promise
- async/await
- 设计模式
- 项目中最难的问题如何解决
- PWA（service worker，IndexedDB，manifest，push notification）
- 测试框架
- Typescript
- css基础：position
- JS基础
- 怎么理解多线程
- await和sleep区别
- 数据库同步，升级
- 英文阅读
- Android线程通信
- 项目框架