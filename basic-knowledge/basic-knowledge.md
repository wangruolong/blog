---
title: 前端基础技能
author: 大大白
date: 2018-11-03 12:00:00
categories:
- 前端基础
tags: 
- 前端基础
---

前端技能的积累，以后需要可以来这里查找。

<!-- more -->
## path
- npm install path
- __dirname：指向当前执行的js的文件的路径。
- 连接路径：path.join([path1][, path2][, ...])
- 路径解析：path.resolve([from ...], to)

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