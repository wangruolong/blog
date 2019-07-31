# immutable使用总结
梳理了使用immutable过程中遇到的问题。

## `state.set('key',value)`
- 如果value是基础类型，比如number，string等，建议可以直接这样set进去。
- 如果value是对象类型，比如map，list等，可以fromJS(value)再设置进去。
- 因为`state.set('key',fromJS(1))`和`state.set('key',1)`，在`state.get('key')`是一样的。但是，`state.set('key',fromJS({a:1}))`和`state.set('key',{a:1})`，在`state.get('key')`是不一样的。前者get出来的是一个immutable对象，后者get出来的是一个js对象。
- 所以综上所述，在set的时候，如果是基础类型可以直接set，如果是对象类型要先`fromJS()`再set进去。
- 以此类推`state.merge({'key2',value2})`这个value也是一样的道理。

## 注意事项
- 不要在container里面toJS()，因为toJS()之后会是一个新的对象，导致react重新render
- 