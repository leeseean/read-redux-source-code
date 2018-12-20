## 概述

applyMiddleware是一个enhancer，可作为createStore的第三个参数，用来增强store。
比如redux-thunk，其源码
```
export default function thunkMiddleware({ dispatch, getState }) {
  return next => action =>
    typeof action === 'function' ?
      action(dispatch, getState) :
      next(action);//next对应下面的store.dispatch
}
```


## 源码

```
export default function applyMiddleware(...middlewares) {
  //createStore作为参数，重写createStore
  return createStore => (...args) => {
    const store = createStore(...args) //获取store
    //重写dispatch
    let dispatch = () => {
      throw new Error(
        `Dispatching while constructing your middleware is not allowed. ` +
          `Other middleware would not be applied to this dispatch.`
      )
    }

    const middlewareAPI = {
      getState: store.getState,//action方法参数
      dispatch: (...args) => dispatch(...args)
    }
    //如thunkMiddleware，接收{getState, dispatch}参数。所有中间件都需此参数
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch) //重写dispatch

    return {
      ...store,
      dispatch
    }
  }
}

```