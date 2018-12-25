## 前言

bindActionCreator的作用是返回包裹dispatch的函数可以直接使用。
一般用在mapDispatchToProps里。

## 源码

```
//将actionCreator转化成dispatch形式
function bindActionCreator(actionCreator, dispatch) {
  return function() {
    return dispatch(actionCreator.apply(this, arguments))
  }
}
/*
参数actionCreators是一个对象或者函数，dispatch即store.dispatch
*/
export default function bindActionCreators(actionCreators, dispatch) {
  //如果传入是函数，直接返回一个包裹dispatch的函数
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }
  //传入参数不符合规范情况
  if (typeof actionCreators !== 'object' || actionCreators === null) {
    throw new Error(
      `bindActionCreators expected an object or a function, instead received ${
        actionCreators === null ? 'null' : typeof actionCreators
      }. ` +
        `Did you write "import ActionCreators from" instead of "import * as ActionCreators from"?`
    )
  }
  //传入参数是Object的情况，遍历对象，根据相应的key，生成包裹dispatch的函数
  const keys = Object.keys(actionCreators)
  const boundActionCreators = {}
  for (let i = 0; i < keys.length; i++) {
    const key = keys[i]
    const actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  // 返回每个value都是包裹dispatch函数的对象
  return boundActionCreators
}
```