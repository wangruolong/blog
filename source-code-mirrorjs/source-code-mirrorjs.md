---
title: mirrorjs源码解读
author: 大大白
date: 2019-12-24 15:55:00
categories:
- WEB前端
- 源码解读
tags:
- mirrorjs
---

React 全家桶提供给了我们一套较为成熟的解决方案。可以应对各种复杂前端 web 系统。
但是当你使用了一阶段只有会发现，许多时候都在机械劳作和编写模板代码。
针对这种情况，我们是否有办法把机械劳作和模板代码交给程序去做，我们只要关心业务如何实现就可以了？
mirror 就是在这种情况下产生的，它封装了 Redux 数据流的过程，给开发人员提供了非常大的便利。

<!-- more -->

## 背景介绍

一般情况下，开发一个登陆页面，需要涉及到以下几个文件。
用来存放登陆页面的输入框按钮的页面 login.js
登陆页面需要调用的方法 loginSmart.js
方法的名称 loginAction.js
方法的类型 actionTypes.js
中间件监听 rootSaga.js
中间件具体执行 loginSaga.js
接口调用 loginService.js
存储登陆的信息 loginReducer.js

<img src="redux_flow.png" />

思考几个问题

1. model 中的 reducers 和 effects 里面的 function 究竟是 action 还是 reducer？
   action 里面函数的作用是 dispatch actionType 到 redux 中，而 reducer 里面函数的作用是接收这个 actionType 并返回新的 store 对象。actions.modelName.xxx 通过某种方式进入到了这里，所以这里更像是 saga 的中间层。
2. model 这种写法是使用了 js 里面什么设计模式？充分利用了 js 的什么特性？
   使用了模块模式。充分利用了 js 闭包的特性。
3. 使用 actions.modelName.xxx 是如何进到这里的？为什么按照 model 这种写法之后，我就可以用 actions.modelName.xxx 去调用 model 里面的方法了。
   分两种情况，一种是 effects，另外一种是 reducers，两个的触发过程是不一样的。
4. reducer 是什么东西？为什么叫做 reducer？reducer 有什么注意事项？
   reducer 就是一个函数，接收旧的 state 和 action，返回新的 state。之所以称作 reducer 是因为它将被传递给 Array.prototype.reduce(reducer, ?initialValue) 方法。保持 reducer 纯净非常重要。永远不要在 reducer 里做这些操作：修改传入参数；执行有副作用的操作，如 API 请求和路由跳转；调用非纯函数，如 Date.now() 或 Math.random()。
5. hook 是怎么实现做到拦截全局 action 的？
   在需要的地方挂上钩子，在 middlewave 把钩子一个个拿出来执行一遍。
6. effects 和 reducers 有什么区别？都写到 effects 里面可不可以？effects 和 reducers 都是利用了 js 的什么特性？
   effects 有经过 middlewave，reducers 没有经过 middlewave。可以是可以，但是不建议这么做。利用了闭包的特性

## 源码解读

推荐 clone 我的源码https://github.com/wangruolong/web-front-end/tree/master/source-code-mirrorjs
里面有大量的注释和精心制作的运行结果，可以助你快速掌握！

<img src="代码结构.png" />

### model.js

作用：把 reducer 和 effects 转换成 redux 能识别的 reducer

1. 检查数据格式，过滤掉非函数的键值对
2. 给 reducer 加上模块名，保证 reducer 全局唯一，返回一个 map 对象
3. 把 initialState 带上，拼接成一个完整的 action2reducer
4. dispatch type 为 todos/test2 的 action
5. combineReducers，什么是 reducer？或者说 reducer 是由什么构成的？

### actions.js

作用：把 reducer 和 effects 转换成 redux 能识别的 action

1. 验证并初始化对象，防止出现 undefined
2. 循环 reducers 把模块名和方法名生成 action 并添加到全局 actions
3. 循环 effects 把模块名和方法名生成 action 并添加到全局 actions 和全局的 effects

### middleware.js

作用：执行全局范围内的 effects，执行全局的钩子 hooks

1. 参数为什么叫做 next？next 就是 dispatch
2. 默认执行其他中间件传递过来的 action
3. 如果 effects 里面有对应的 actionType，则 effects 里面的 action 优先级更高
4. 执行钩子里面的函数，并把 action 和 getState 传递给他们

<img src="redux_middlewave.png" />

### effects.js

作用：传值赋值与引用赋值。

JS 中的类型只有 6 种，其中基本数据类型有 5 种分别为 string，number，boolen，null，undefined。引用类型有一种，就是 object。object 是一个大的综合体。在 JS 中除了那 5 个基本数据类型以外，其他的一切皆对象。

### hooks.js

作用：在全局范围内拦截 action

1. 用个钩子把函数勾起来
2. 把钩子一个个拿起来看看
3. 把第二个钩子删掉
4. 把钩子一个个拿起来看看

## 总结

### 闭包

hooks、effects 中使用的闭包

1. 闭包是函数和声明该函数的词法环境的组合。
2. 闭包是内部函数在其父函数已经终止之后可以引用其外部封闭函数中存在的变量的一种手段。
3. 闭包就是能够读取其他函数内部变量的函数。例如在 javascript 中，只有函数内部的子函数才能读取局部变量，所以闭包可以理解成“定义在一个函数内部的函数”。在本质上，闭包是将函数内部和函数外部连接起来的桥梁。
4. 定义在一个函数内部的函数。父函数终止后引用父函数的内部变量。记住这句话：在这个闭包区间内该变量永久有效。
