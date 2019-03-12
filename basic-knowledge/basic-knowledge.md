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

##  `presets`
`presets`字段设定转码规则，官方提供以下的规则集，你可以根据需要安装。

```
# 最新转码规则
$ npm install --save-dev babel-preset-latest

# react 转码规则
$ npm install --save-dev babel-preset-react

# 不同阶段语法提案的转码规则（共有4个阶段），选装一个
# Stage 0 - Strawman（展示阶段）
# Stage 1 - Proposal（征求意见阶段）
# Stage 2 - Draft（草案阶段）
# Stage 3 - Candidate（候选人阶段）
# Stage 4 - Finished（定案阶段）
$ npm install --save-dev babel-preset-stage-0
$ npm install --save-dev babel-preset-stage-1
$ npm install --save-dev babel-preset-stage-2
$ npm install --save-dev babel-preset-stage-3
```
```json
{
  "presets": [
    "env",
    "es2015",
    "react",
    "stage-2"
  ]
}
```
如果presets没有设置`stage-2`，因为这里面有些语法还在`stage-2`阶段还没有正式发布，所以如果没有加上这个配置就无法解析这种语法。
<img src="presets没有设置stage-2.png" />


## `plugins`

`plugins`字段设定插件
- 当项目启用generate的时候报错`regeneratorRuntime is not defined`，则需要安装`babel-plugin-transform-runtime`插件
- transform-decorators-legacy
- add-module-exports
```json
{
  "plugins": [
    "add-module-exports",
    "transform-runtime"
    "transform-decorators-legacy",
  ]
}
```

## 在webpack中
`Using removed Babel 5 option`报错使用了被移除的babel5中的语法，是因为没有把`node_modules`排除掉。目录是相对package.json的路径。
因为`node_modules`里面有一些包是用了Babel 5中的语法，但是在这个loader里面又是被移除了，所以就报错了，根本解决办法就是把`node_modules`exclude掉。
```js
   module: {
        rules: [
            {
                exclude:/node_modules/,
                test: /\.(js|jsx)$/,
                loader: ['babel-loader','eslint-loader']
            }
        ]
   }
```
<img src="未exclude导致的babel报错.png" />

# eslint
- package.json
```json
{
"babel-eslint": "^10.0.1",
"eslint": "^5.13.0",
"eslint-plugin-react": "^7.12.4",
"eslint-loader": "^2.1.1",
}
```
## eslintrc.json
特别要注意parser要设置babel-eslint，用babel转换之后给eslint验证规则。
`babel-eslint`，`eslint`，`eslint-plugin-react`主要用于eslintrc.json的配置。
```json
{
  "parser": "babel-eslint",
  "plugins": ["react"],
  "env": {
    "browser": true,
    "commonjs": true,
    "es6": true
  },
  "extends": ["plugin:react/recommended"],
  "settings": {
    "react": {
      "createClass": "createReactClass",
      "pragma": "React",
      "version": "detect",
      "flowVersion": "0.53"
    },
    "propWrapperFunctions": [
      "forbidExtraProps",
      { "property": "freeze", "object": "Object" },
      { "property": "myFavoriteWrapper" }
    ]
  },
  "parserOptions": {
    "ecmaFeatures": {
      "jsx": true
    },
    "ecmaVersion": 2018,
    "sourceType": "module"
  },
  "rules": {
    "indent": ["error", "tab"],
    "linebreak-style": ["error", "windows"],
    "quotes": ["error", "single"],
    "semi": ["error", "never"]
  }
}

```
## webpack.js
`eslint-loader`用于webpack的配置。
```js
module: {
		rules: [
			{
				exclude: /node_modules/,
				test: /\.(js|jsx)$/,
				loader: ['babel-loader','eslint-loader']
			}
    ]
}
```
## 增加钩子让用户在提交前都执行以下eslint。
Git hooks made easy
在package.json增加husky依赖，同时配置commit之前和push之前需要执行的命令，强制用户执行eslint检查。
```json
{
  "husky": {
    "hooks": {
      "pre-commit": "npm test",
      "pre-push": "npm test",
      "...": "..."
    }
  },
  "devDependencies":{
    "husky": "^1.3.1",
  }
}
```

# webpack
## 样式表的Loader`style-loader ``css-loader``sass-loader`
- style-loader把css放到<styles/>里面，而css-Loader则是把css通过<link/>引入。
- 启用MiniCssExtractPlugin.loader会把css进行压缩。
- `css-loader`要启用modules=true，在代码里面才能import styles from 'style.css'。同时设置localIdentName规则把css的名称进行干扰，防止全名冲突。
```js
{
  loader: 'css-loader',
  options: {
    modules: true,
    localIdentName: '[path][name]__[local]--[hash:base64:5]'
  }
}
```
<img src="未开启modules.png" />
<img src="开启modules.png" />

## IP访问
在package.json的执行脚本中增加`--host 0.0.0.0`就可以通过ip访问了，缺点是一开始启动的时候无法打开网页。
也可以通过js获取本机ip
<img src="获取ip.png" />

## `file-loader`与`url-loader`
- 区别
  + 相同点：都可以用来加载资源文件。
  + 不同点：`url-loader`可以设置小于指定大小的文件直接打包到js里面，减少请求次数。
- 联系
  + 当有文件超过`url-loader`指定的文件大小后，不会被打包到js里面，但是它就变成需要`file-loader`加载否则会报错。
  + 把小图片打包到js里面减少请求次数各有利弊。优点：可以减少小图片的请求次数，降低网络IO的请求次数。缺点：图片会被转换成base64的格式和js一起打包进入会带出新的问题，就是这样会导致js越来越大，这样加载单个js可能需要的时长会更长。另外，base64的算法是把原来的数据每3位用4位替换，这样原来如果是1，就会变成4/3，相当于变大了4/3倍。
  + 因此是否需要使用`url-loader`把小图片打包到js文件需要权衡后再做决定。把小图片打包到js需要做的牺牲就是js文件会变大。
  + `file-loader`实现的是懒加载，只有在页面需要用到具体的元素才会加载，否则并不会加载。这样能提高整体的性能。
- 核心用法
  + `file-loader`的`publicPath`属性，用来指定访问的路径。
  + `file-loader`的`outputPath`属性，用来指定打包输出的路径和访问的路径。建议使用outputPath属性，因为这个属性同时指定了打包输入和访问的路径，而`publicPath`只指定了访问的路径，如果你实际打包的路径不是这个就访问不到了。
  + `url-loader`的`limit`属性，用来指定小于限定的字节(Byte)则打包到js文件里面，超过限定的字节(Byte)则需要`file-loader`加载。

# React
## react router
未装propTypes报错`Cannot read property 'array' of undefined`
react15之后prop-types被剥离开来，而react-router里面的很多写法还是react.proptypes这样肯定报错。所以有两种方案，一种是把react降到15之前（不包括15），另外一种就是把react-router升级到3.x以上版本。为什么不直接升级到4.x因为我试用了一下发现是服务端渲染，而且一大堆配套的都要升级，因此升级到3.x是最明智的选择。
<img src="未装propTypes报错1.png" />
<img src="未装propTypes报错2.png" />

## redux saga
当我dispatch一个action后，这个先发到reducer
然后才走到saga，saga拿到后put了一个新的action这个新的action才是我们要处理的。
简单来说，当我们dispatch一个action后，先发到reducer，然后saga同时也收到了一份，这时候saga可以put出新的action给reducer接收。






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