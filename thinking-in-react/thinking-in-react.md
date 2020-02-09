---
title: Thinking in React
author: 大大白
date: 2019-11-06 16:55:00
categories:
- WEB前端
- 最佳实践
tags:
- React
---

使用 React 开发过程中的一些最佳实践。持续更新...

<!-- more -->

## 路由懒加载

组件级别的代码分隔 react-loadable
React.Lazy()

## smart & dump

区分容器 smart 和组件 dump 的概念。

## keys & props

推荐 clone 我的源码https://github.com/wangruolong/web-front-end/tree/master/thinking-in-react
里面有大量的注释和精心制作的运行结果，可以助你快速掌握！

### 引入问题

1. 我们经常需要对一个数组进行循环，渲染一个列表。但是控制台经常会报警告 Warning: Each child in a list should have a unique "key" prop.有没有思考过这是为什么？
2. 遇到这种问题的一般做法是直接把 index 赋值给 key。这样会有什么问题？
3. 重新 render 的时候出问题了。为什么会这样？

### 分析问题

1. 不加 key 的情况下，react 分不清哪个是哪个。
2. React 在循环的时候自动帮元素加上序号。
3. diff 算法的比较规则，优先比较 key，如果 key 不存在则比较组件名称，如果组件名称一样则 react 认为是同一个组件，不 unmount 直接重新 render。至于 props 里面的数据一不一样，React 认为这只是 props 数据变化了而已。
   （React diff 算法可以参考这篇文章 https://www.jianshu.com/p/3ba0822018cf）

4. 不加 key 的情况下，react 会默认加上 key，当对比到同一个 key 的组件名称不一致，就直接销毁掉了。
5. 加 key 后，react 会根据 key 进行比较，同一个 key 如果组件名称也一样，则认为是同一个组件不 unmount，而是重新 render。

### 解决问题

1. 为什么 input 的值会没掉？
2. input 有没有重新 render？但是有没有被销毁？为什么？
3. 因为 input 的值既不是受控也不是非受控，他的值并没有被 react 维护。而值被清空了并不是说组件被销毁了，而是说重新 render 了，但是值没有被保留下来。

4. 非受控组件。this.state.value 并没有受到 props 的控制，所以当重新 render 后，input 还是由原来的值控制。
5. 只不过 props 改变了，这个本质上和原来是同一个组件，只不过 props 变了而已。

## hooks

- 为了处理在触发的时候还不存在，等到执行的时候才存在的对象。

场景：一个搜索组件 Search，一个文章详情组件 Article，搜索结果返回用 i 标签包裹的关键词。
需求：搜索文章关键词，点击结果文章自动滚动到第一个关键词。
实现方式：在渲染结束后，真实 Dom 上面查找 i 节点，找到 i 节点后获取它的 offset 然后设置 Article 的 scrollTop。
这就要求必须要有真实 dom 的时候才搜索，所以要写在 didUpdate，写在 didMount 也可以但是它只会触发一次所以最终还是得写在 didUpdate。
didUpdate 会被很多种情况触发，常规的做法是在 didUpdate 里面设置各种属性判断，但是这样会导致项目越来越难维护。didUpdate 里面应该尽量简单。

用钩子，可以在点击的那个地方挂上一个钩子，然后在 didUpdate 那边不断循环这些钩子，一旦钩子被执行就直接销毁，这样也不会有副作用。
相当于在点击的时候 producer，然后在执行的地方 consumer，执行之后把相应的钩子销毁掉就行了。

```js
// 钩子数组，我有很多把钩子。
const hooks = [];
// 一根钩子抹点油。怎么样？我就问你这种注释看得懂吗？
function hook(subscriber) {
  if (typeof subscriber !== "function") {
    throw new Error("Invalid hook, must be a function!");
  }
  const subscriberIndex = hooks.indexOf(subscriber);
  if (subscriberIndex == -1) {
    hooks.push(subscriber);
    return () => {
      const result = hooks.splice(hooks.indexOf(subscriber), 1);
      return result[0];
    };
  } else {
    return () => {
      const result = hooks.splice(subscriberIndex, 1);
      return result[0];
    };
  }
}
// 篮子
const baskets = {};
// 出钩
function goHook(key, func) {
  baskets[key] = hook(func);
}
// 收钩。把参数带上。
// 这个...args的作用是【rest】，接收剩余参数。
function backHook(key, ...args) {
  try {
    if (baskets[key]) {
      console.log("找到钩子", baskets[key]);
      // 这个...args的作用是【spread】，把对象展开。
      baskets[key]()(...args);
      delete baskets[key];
    } else {
      console.log("没找到钩子");
    }
  } catch (e) {
    console.log(e);
  }
}

export { hooks, baskets, goHook, backHook };
```

## IoC

- 这是为了解决跨多层级组件之间的通讯的问题。组件的数据没有在 redux 中维护，而是在 state 中。组件内部特定的方法。

场景：文章列表组件 Articles，他们归属于同一个分类。点击分类按钮，打开侧边栏分类列表组件 Categories，分类列表顶部有一个搜索框 Input 用来搜索分类，点击分类 CategoryItem 后刷新文章列表。
需求：选中某个分类，关闭侧边栏，清空 Input 框。
这要求 Articles、Input 的数据都要从 redux 存取。当选中 CategoryItem 后，同时设置 Articles 和 Input 的数据。
但是 Input 只不过是一个普通的输入框却要绕一大圈到 redux 里面，如果 Input 不在 redux 存储，这个需求将很难完成。
这时候 IoC 体现出了价值，它可以把 Categories 放入 IoC 容器中，在 CategoryItem 里面去调用 Categories 的方法来清空 Input。
因为在实际开发中 Input 受控方式直接在 state 是最方便的。

总的来说，适用于以下这种场景。由 state 控制的，或者是组件内部的方法，没办法通过 callback，或者修改 redux 就可以实现的。

比如一个 List 组件，里面的数据已经是一个完整的对象，当打开单个 Item 的时候不需要再请求一次，因此这个 Item 的 data 经常会被放在 state 维护。
但是当 Modal 弹出框后需要修改这个 data，问题就出现了。因为你虽然更新了 redux，但是当前 modal 里面的数据却是在父容器的 state 中，所以这时候无法和新数据保持一致。
当然这种情况可以用 callback 的方式解决，就是当更新完数据之后调用一个 callback 把最新的值 set 回来。

```js
import emitter from "utils/eventUtil";
const IoC = {};
/**
 * 发布订阅模式
 * 可以用来处理setState，但是不能用来处理调用对象的方法。这个还需完善，可以考虑用IOC容器。
 * @param {*} subscriberKey
 */
function PubSub(subscriberKey) {
  return WrappedComponent => {
    return class extends WrappedComponent {
      constructor(props) {
        super(props);
      }
      componentDidMount() {
        super.componentDidMount && super.componentDidMount();
        this.eventEmitter = emitter.addListener(
          subscriberKey,
          this.apiSetState
        );
        IoC[subscriberKey] = this;
      }
      apiSetState = ({ key, value }) => {
        super.setState({
          [key]: value
        });
      };
      componentWillUnmount() {
        super.componentWillUnmount && super.componentWillUnmount();
        emitter.removeListener(subscriberKey, this.apiSetState);
      }
    };
  };
}

export default PubSub;
export { IoC };
```

## 设计原则

### 单一职责原则

### 接口隔离原则

### 里氏替换原则

### 开闭原则

### 依赖倒置原则

getData()、renderItem()调用外部的方法。实现的方式依赖于抽象而不是具体。

### 组合/聚合复用原则

## 设计模式

### 单例模式

由于所有的组件都要互相通讯，所以必须保证所有的组件都在同一个容器中，所以使用单例模式创建 IoC 容器。

```js
function Sequence() {
  if (!Sequence.instance) {
    var privateIndex = 0;
    this.next = () => {
      return ++privateIndex;
    };
    Sequence.instance = this;
  }
  return Sequence.instance;
}
```

### 装饰器模式

高阶函数，返回函数的闭包。

发布-订阅机制

event.js

```js
import { EventEmitter } from "events";
export default new EventEmitter();
```

pubsub.js

```js
import emitter from "utils/eventUtil";
function PubSub(subscriberKey) {
  return WrappedComponent => {
    return class extends WrappedComponent {
      constructor(props) {
        super(props);
      }
      componentDidMount() {
        super.componentDidMount && super.componentDidMount();
        this.eventEmitter = emitter.addListener(
          subscriberKey,
          this.apiSetState
        );
      }
      apiSetState = ({ key, value }) => {
        super.setState({
          [key]: value
        });
      };
      componentWillUnmount() {
        super.componentWillUnmount && super.componentWillUnmount();
        emitter.removeListener(subscriberKey, this.apiSetState);
      }
    };
  };
}

export default PubSub;
```

控制反转机制

```js
// 容器
const Ctx = {};
// 把组件注入到容器
function Ctrl(subscriberKey) {
  return WrappedComponent => {
    return class extends WrappedComponent {
      constructor(props) {
        super(props);
      }
      componentDidMount() {
        super.componentDidMount && super.componentDidMount();
        Ctx[subscriberKey] = this;
      }

      componentWillUnmount() {
        super.componentWillUnmount && super.componentWillUnmount();
      }
    };
  };
}

export { Ctrl, Ctx };
```

### 代理模式

就类似 AOP 切面编程。

### 中介者模式

通过 dependencies 把状态传出去。

### 发布-订阅模式和观察者模式的对比

### 模块模式

充分利用了闭包的特性

## 待完善

- Thinking in React 思考如何加入 AOP，是否能和 hooks 结合？
- 跟分块相关的，可以用webpack，也可以用第三方包。
  - 掌握 react-loadable 和 React.Lazy()
  - 掌握webpack分chunk打包
- 