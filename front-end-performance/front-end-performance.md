---
title: 前端性能优化
author: 大大白
date: 2017-11-03 12:00:00
categories:
- 性能优化
tags: 
- 性能优化
---

前端性能优化，根据不同的场景和目标有多种方式可以进行优化。

<!-- more -->

## webpack
- 压缩
- 切割成多个chunk 
- 懒加载

### HMR
模块热替换(Hot Module Replacement 或 HMR)是 webpack 提供的最有用的功能之一。它允许在运行时更新各种模块，而无需进行完全刷新。
https://www.webpackjs.com/concepts/hot-module-replacement/

### tree shaking
tree shaking 是一个术语，通常用于描述移除JavaScript 上下文中的未引用代码(dead-code)。
```package.json
{
  "name": "your-project",
  "sideEffects": [//需要被排除在外的，就算有无用代码也不会去掉，一样会被编译进去。
    "./src/some-side-effectful-file.js",
    "*.css"
  ]
}
```
### shimming 全局变量
webpack 编译器(compiler)能够识别遵循 ES2015 模块语法、CommonJS 或 AMD 规范编写的模块。然而，一些第三方的库(library)可能会引用一些全局依赖（例如 jQuery 中的 $）。这些库也可能创建一些需要被导出的全局变量。这些“不符合规范的模块”就是 shimming 发挥作用的地方。
```webpack.config.js
plugins: [
        new webpack.ProvidePlugin({
            // _: 'lodash',//把lodash赋予全局变量_
            join: ['lodash', 'join']//把lodash.join赋予全局变量join
        })
    ],
```

### 懒加载
懒加载或者按需加载，是一种很好的优化网页或应用的方式。这种方式实际上是先把你的代码在一些逻辑断点处分离开，然后在一些代码块中完成某些操作后，立即引用或即将引用另外一些新的代码块。这样加快了应用的初始加载速度，减轻了它的总体体积，因为某些代码块可能永远不会被加载。
```src/print.js
console.log('The print.js module has loaded! See the network tab in dev tools...');

export default () => {
  console.log('Button Clicked: Here\'s "some text"!');
}
```
```src/inxdex.js
import _ from 'lodash';
function component() {
    var button = document.createElement('button');
    var br = document.createElement('br');
    button.innerHTML = 'Click me and look at the console!';
    element.appendChild(br);
    element.appendChild(button);
    
    button.onclick = e => import(/* webpackChunkName: "print" */ './print').then(module => {
        var print = module.default;
        print();
    });
    
    return element;
}
document.body.appendChild(component());
```

### css抽取
它会将所有的入口 chunk(entry chunks)中引用的 *.css，移动到独立分离的 CSS 文件。因此，你的样式将不再内嵌到 JS bundle 中，而是会放到一个单独的 CSS 文件（即 styles.css）当中。 如果你的样式文件大小较大，这会做更快提前加载，因为 CSS bundle 会跟 JS bundle 并行加载。
https://github.com/webpack-contrib/mini-css-extract-plugin
```
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const devMode = process.env.NODE_ENV !== 'production'

module.exports = {
  plugins: [
    new MiniCssExtractPlugin({
      // Options similar to the same options in webpackOptions.output
      // both options are optional
      filename: devMode ? '[name].css' : '[name].[hash].css',
      chunkFilename: devMode ? '[id].css' : '[id].[hash].css',
    })
  ],
  module: {
    rules: [
      {
        test: /\.(sa|sc|c)ss$/,
        use: [
          devMode ? 'style-loader' : MiniCssExtractPlugin.loader,
          'css-loader',
          'postcss-loader',
          'sass-loader',
        ],
      }
    ]
  }
}
```

### 分离 应用程序(app) 和 第三方库(vendor)
- `venders`用来打包第三方库。但是如果只是配置这个，app.js并没有变小，因为这只是新打包了一个venders.js文件，本身和app.js没什么关联。
- `venders`必须要配合`optimization.splitChunks`才能实现对app.js里面第三方库的分离。
- `optimization.splitChunks`用来剥离充重复引用的包。简单来说就是把app.js和venders.js里面重复引用的包分离出来到common.js，然后在app.js和venders.js留下一个require入口指向common.js。举个例子：index.js中有a.js和b.js，然后a.js引入了xxx.js，b.js也引入了xxx.js，这样如果直接打包xxx.js会被打包两次，所以通过`optimization.splitChunks`的配置， xxx.js只被打包了一次。
- 这样通过`venders`和`optimization.splitChunks`的配合就能把一个比较大的app.js文件变成app.js、venders.js、common.js这几个比较小的文件。

```javascript
entry: {
		app: './src/index.js',
		venders:['react-dom','iconv-lite','immutable','moment','crypto-js']
	},
optimization: {
		splitChunks: {
			cacheGroups: {
				commons: {
					name: 'vender',
					chunks: 'initial',
					minChunks: 2
				}
			}
		}
	},
```

### import方式的不一样
- `import uniqBy from 'lodash/uniqBy'`：只引入了lodash里面的uniqBy方法。
- `import { uniqBy} from 'lodash'`：先把lodash整个对象引入后，再引入uniqBy方法。
最好是第一种不用引入lodash整个对象，只要把相关的函数引入就可以，这样体积可以小很多。
