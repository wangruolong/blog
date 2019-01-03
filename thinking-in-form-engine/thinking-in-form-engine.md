---
title: 表单引擎设计思想
author: 大大白
date: 2018-06-28 16:48:00
categories:
- 表单引擎
tags: 
- 设计思想
---

# 概述
Formtastic表单引擎是一个采用了单向数据流，双向数据绑定，三大事件机制，四大核心驱动，打通五脏六腑七经八脉，按照九大设计模式为指导思想的十全十美的表单引擎。

本次的分享将带你走进Formtastic的世界，为你阐述其设计思想，让你领悟其设计精髓，为Formtastic正名！

<!-- more -->

## 用过Formtastic的人的评价
- 看了一个礼拜还是一头雾水
- 说是三分钟入门，我看了三个月还不懂这个东西是怎么回事
- 看到Formtastic就害怕
- 那我只能表示遗憾了
- 离职
- 离职
- 离职
- 离职
- ...

## 为什么会这样
- 看到我弄了一个模板配置表单的感觉好复杂；
- 然后你看到后，也赶紧弄出了一个更加复杂的数据结构让两个加起来更加复杂；
- 然后他看到你做的这些后，赶紧加入了saga并且封装了一些没几个人能看得懂的方法和工具；
- 然后我看到他做的这些后，赶紧加入了中介者dependencies；
- 然后他看到后，赶紧加入了logic；
- 然后你看到后又加了courier；
- 于是这个引擎越来越复杂。。。

就这样我做的你没看懂，你做的他没看懂，他做的我也没看懂。好在最终经过不断的磨合，所有的东西还是可以通过一条主线串起来。我们原来只要理解自己做的那部分就可以，但是你们却需要理解所有人做的东西。只有我们自己心里清楚，这整个是怎么串起来的。

## 取其精华去其糟粕
###  设计模式
- 创建型
    + 单例模式
- 结构型
    + 适配器模式
    + 过滤器模式
    + 组合模式
    + 装饰器模式
    + 外观模式
    + 代理模式
- 行为型
    + 解释器模式
    + 中介者模式
    + 观察者模式
    + 策略模式
    + 访问者模式

### 控制反转（Inversion of Control）
- 依赖注入
- 依赖查找

```javascript
class IoC {
    constructor() {
        this.context = {}
    }
    static getInstance() {
        if (!this.instance) {
            this.instance = new IoC()
        }
        return this.instance
    }
    injection(key, component) {
        this.context[key] = component
    }
    lookup(key) {
        return this.context[key]
    }
}
```

### 面向切面（Aspect-Oriented Programming）
面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。
- 预编译：编译时把当前的方法放入AOP中。
- 运行时：运行时AOP中会动态的把需要的方法织入。
```javascript
 <RadioGroup options={options}
             onChange={this._aop.bindArgs(this._onChange, this, { target: { value: !this.state.value } })}
             value={this.state.value} />
```

### 数据流驱动与模板驱动
```javascript
const config = {
    AOP: true
}
let eventConfig = {
    source: 'StandardCase.EditMode',//源组件
    target: 'Rate.EditMode',//目标组件
    sourceFunc: 'onChange',//源方法
    targetFunc: 'onChange',//目标方法dynamicFunc
    targetFuncArgs: [ '4' ],//参数
    triggerMode: 'custom',//触发方式：static静态触发，按照预设参数去触发；dynamic动态触发，在运行时根据触发的方法带过来的参数触发；custom自定义的，按照自定义的规则触发。
    DSL: '(function (args){if(args===true){return 4} else {return 2}})'//自定义方法
}
```
### 形成的规范
- 组件必须装饰@FormtasticMini，必须继承BaseComponent。
- 组件必须以函数驱动为导向，出入格式要规范参数全部用数组传递等等。
- 每个组件都必须提供自己的API列表，方便其他开发者配置和使用。

### 基类BaseComponent
- 实例化IoC容器，并且把组件注入到容器中。
- 绑定aop切面，让子类可以方便的使用aop切面。

```javascript
export class BaseComponent extends Component {
    static displayName = 'BaseComponent'
    constructor(props) {
        super(props)
        this.IoC = IoC.getInstance()
        this._aop = bind(this.aop, this)
    }
    componentWillMount() {
        if (this.props.displayName && this.props.displayName !== 'undefined') {
            if (!(this.props.displayName.indexOf('FormtasticMini') > -1)) {
                this.IoC.injection(this.props.displayName, this)
            }
        }
    }
    aop(...args) {
        return this.props.AOP(...args)
    }
}
```

### 核心类FormtasticMini
- 为什么要使用反向继承？
    + 从宏观的角度来看
        * 继承子组件的属性，并且可以控制子组件。
    + 从微观的角度来看
        * 继承了子组件就可以把AOP传给子组件，子组件可以使用AOP进行切面绑定。但是这个作用，属性代理也可以实现。
        * 只有反向继承子组件，才能使用子组件继承自BaseComponent的IoC容器，然后再通过IoC容器控制其他组件。针对这一点，属性代理无法实现。如果不反向继承，这将无法获得IoC容器。
        
        * 使用反向继承的前提下，不使用this.IoC，而是使用this.props.IoC可不可以？
        不可以。这里的this指向EC，不是指向子组件。EC的props里面并没有IoC容器，EC的IoC容器是继承自子组件子组件继承自BaseComponent的IoC容器。
        
        * 使用属性代理的前提下，this.IoC和this.props.IoC有什么区别？
         属性代理没有继承子组件所以没有this.IoC，同时也没有人传递给他props，所以这里的this.props.IoC也是undefined

        * 使用反向的前提下，this.IoC和this.props.IoC有什么区别？
         反向继承了子组件子组件继承了BaseComponent，所以EC是有this.IoC的。但是，这里的this指向的是EC并不是指向子组件，所以this.props.IoC并没有值。

    + 综上所述。这里一定要用反向继承，并且一定要用this.IoC来获取容器以此来控制组件。


```javascript
export function FormtasticMini(WrappedComponent) {
    let ProxyComponent = new Proxy(WrappedComponent, handler)
    return class EC extends WrappedComponent {
        static displayName = `FormtasticMini(${getDisplayName(WrappedComponent)})`
        constructor() {
            super()
        }
        //如果需要对子组件的方法进行切面，需要显示调用此方法。
        //注意这里的this是EC组件return之前的WrappedComponent，所以这里的this指向WrappedComponent但是它（参考对象是EC组件）没有props的。
        //this2是子组件的this，指向真实的子组件，所以有props，它与BaseComponent类似都是经历了React生命周期后形成的。
        AOP(func, self, ...args) {
            //在类的加载、实例化的时期执行。
            // return () => {
            //在类的运行时期执行。
            if (config.AOP) {
                if (this.displayName === eventConfig.source) {
                    if (func.name && func.name.indexOf(eventConfig.sourceFunc) > -1) {
                        let toolInstance = this.IoC.lookup(eventConfig.target)
                        if (toolInstance && typeof toolInstance[eventConfig.targetFunc] === 'function') {
                            if (eventConfig.triggerMode === 'static') {//static静态触发，按照预设参数去触发
                                let event = args.pop()//触发的事件
                                toolInstance[eventConfig.targetFunc](...eventConfig.targetFuncArgs)//触发目标组件方法
                                func(event)//触发源组件方法
                            } else if (eventConfig.triggerMode === 'dynamic') {//dynamic动态触发，在运行时根据触发的方法带过来的参数触发
                                let params = args[0]
                                toolInstance[eventConfig.targetFunc](params)
                                func(params)
                            } else if (eventConfig.triggerMode === 'custom') {//custom自定义的，按照自定义的规则触发
                                let event = args.pop()//触发的事件
                                let inData = event.target.value
                                let customFunc = eval(eventConfig.DSL)
                                let outData = customFunc(inData)
                                toolInstance[eventConfig.targetFunc](outData)
                                func(event)
                            }
                        }
                    }
                }
            } else {
                func(...args)
            }
            // }
        }
        render() {
            const props = Object.assign({}, this.props, { AOP: this.AOP, IoC: this.IoC, displayName: WrappedComponent.displayName })
            return <ProxyComponent {...props} />
        }
    }
}
function getDisplayName(WrappedComponent) {
    return WrappedComponent.displayName || WrappedComponent.name || 'Component'
}
let handler = {
    construct: function (target, args) {
        // console.log(`construct ${target}!`)
        return Reflect.construct(target, args)
    },
    get: function (target, key, receiver) {
        // console.log(`getting ${key}!`)
        return Reflect.get(target, key, receiver)
    },
    set: function (target, key, value, receiver) {
        // console.log(`setting ${key}!`)
        return Reflect.set(target, key, value, receiver)
    }
}

```

### 应用方
- 加入@FormtasticMini注解，把自己注入到IoC容器中。
- 继承BaseComponent，可以使用ioc和aop

```javascript
@FormtasticMini
export default class EditMode extends BaseComponent {
    static displayName = 'Rate.EditMode'
    static propTypes = {
        submitData: PropTypes.func
    }
    constructor(props) {
        super(props)
        this.state = {
            value: 1
        }
        this._onChange = bind(this.onChange, this)
        this._confirm = bind(this.confirm, this)
    }
    confirm() {
        this.props.submitData(this.state.value)
    }
    onChange(args) {
        this.setState({
            value: args
        })
    }
    dynamicFunc(args) {
        this.setState({
            value: 3
        })
    }
    render() {
        return (
            <div className={styles.container}>
                <div>
                    <Rate allowHalf
                        allowClear
                        value={this.state.value}
                        defaultValue={1}
                        onChange={this._onChange} />
                </div>
                <Button className={styles.confirmBtn}
                    htmlType='button'
                    loading={false}
                    shape={null}
                    icon={null}
                    onClick={this._aop.bindArgs(this._confirm, this)}
                    size='default'
                    type='primary'
                    ghost>
                    确定
                </Button>
            </div>
        )
    }
}
```

```javascript
@FormtasticMini
export default class EditMode extends BaseComponent {
    static displayName = 'StandardCase.EditMode'
    static propTypes = {
        preview: PropTypes.bool,
        toolData: PropTypes.bool,
        submitData: PropTypes.func,
        AOP: PropTypes.func
    }
    constructor(props) {
        super(props)
        this.state = {
            value: true
        }
        this._onChange = bind(this.onChange, this)
        this._confirm = bind(this.confirm, this)
    }
    onChange(e) {
        this.setState({
            value: e.target.value
        })
    }
    confirm() {
        this.props.submitData(this.state.value)
    }
    render() {
        const options = [
            { label: '标准例', value: true },
            { label: '反例', value: false }
        ]
        return (
            <div className={styles.container}>
                <div>
                    <RadioGroup options={options}
                        onChange={this._aop.bindArgs(this._onChange, this, { target: { value: !this.state.value } })}
                        value={this.state.value} />
                </div>
                <Button className={styles.confirmBtn}
                    htmlType='button'
                    loading={false}
                    shape={null}
                    icon={null}
                    onClick={this._confirm}
                    size='default'
                    type='primary'
                    ghost>
                    确定
                </Button>
            </div>
        )
    }
}
```

### 动态加载组件库
- ToolsMng类

```javascript
export default new class ToolsMng {
	constructor() {
		this.toolMap = {}
	}
	registerTool(toolKey, tool) {
		this.toolMap[toolKey] = tool
	}
	getTool(toolKey) {
		return this.toolMap[toolKey]
	}
	getToolMap() {
		return this.toolMap
	}
}
```

> ToolsInit类
```javascript
import toolMng from './toolsMng'
toolMng.registerTool('audio_recorder', require('./audioRecorder'))
```

> 具体创建方式
```javascript
const comp = toolMng.getTool(tool.key, tool.version)//取出配置信息的key
React.createElement(comp || ToolNotFound, props)
```

---
<div align=center><h2>如果觉得我的文章对您有用，请随意打赏。您的支持将鼓励我继续创作！</h2></div>
<div align=center><img src="receiving.jpg" width=300 /></div>