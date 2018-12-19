## 概述

redux中通过createStore方法来创建store以存放应用中所有的state。
```
const store = createStore(reducer, [preloadedState], enhancer);
```
createStore接受3个参数，第一个是reducer，第二个是初始的state，
第三个enhancer是store的增强器。
执行本方法返回一个对象，包含dispatch，subsribe，getState，replaceReducer，observable五个方法。

## 源码

```
export default function createStore(reducer, preloadedState, enhancer) {
  if (
    (typeof preloadedState === 'function' && typeof enhancer === 'function') ||
    (typeof enhancer === 'function' && typeof arguments[3] === 'function')
  ) {
    throw new Error(
      'It looks like you are passing several store enhancers to ' +
        'createStore(). This is not supported. Instead, compose them ' +
        'together to a single function'
    )
  }

  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }

  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }

    return enhancer(createStore)(reducer, preloadedState)
  }

  if (typeof reducer !== 'function') {
    throw new Error('Expected the reducer to be a function.')
  }

  //上面这部分是对参数的一些限制

  let currentReducer = reducer
  let currentState = preloadedState
  let currentListeners = []
  let nextListeners = currentListeners
  let isDispatching = false
  // 在每次修改监听函数数组之前复制一份，实际的修改的是新
  // 复制出来的数组上。确保在某次 dispatch 发生前就存在的监听器，
  // 在该次dispatch之后都能被触发一次。
  function ensureCanMutateNextListeners() {
    if (nextListeners === currentListeners) {
      nextListeners = currentListeners.slice()
    }
  }

  /**
   * 读取当前的state
   */
  function getState() {
    if (isDispatching) {
      throw new Error(
        'You may not call store.getState() while the reducer is executing. ' +
          'The reducer has already received the state as an argument. ' +
          'Pass it down from the top reducer instead of reading it from the store.'
      )
    }

    return currentState
  }

  function subscribe(listener) {
    if (typeof listener !== 'function') {
      throw new Error('Expected the listener to be a function.')
    }

    if (isDispatching) {
      throw new Error(
        'You may not call store.subscribe() while the reducer is executing. ' +
          'If you would like to be notified after the store has been updated, subscribe from a ' +
          'component and invoke store.getState() in the callback to access the latest state. ' +
          'See https://redux.js.org/api-reference/store#subscribe(listener) for more details.'
      )
    }

    let isSubscribed = true  //初始订阅状态为true

    ensureCanMutateNextListeners()  //在每一次调用 dispatch() 之前监听器数组都会被复制一份。
    nextListeners.push(listener)  //将需要监听的方法存入监听器数组
    //返回一个取消订阅的方法
    return function unsubscribe() {
      if (!isSubscribed) {
        return
      }

      if (isDispatching) {
        throw new Error(
          'You may not unsubscribe from a store listener while the reducer is executing. ' +
            'See https://redux.js.org/api-reference/store#subscribe(listener) for more details.'
        )
      }

      isSubscribed = false  //取消订阅后订阅状态为false

      ensureCanMutateNextListeners()  //取消订阅后判断监听器数组是否和当前监听器数组相等，如果相等，则复制一份
      //从监听器数组中删除当前监听的方法
      const index = nextListeners.indexOf(listener) 
      nextListeners.splice(index, 1)
    }
  }

  function dispatch(action) {
    if (!isPlainObject(action)) {
      throw new Error(
        'Actions must be plain objects. ' +
          'Use custom middleware for async actions.'
      )
    }

    if (typeof action.type === 'undefined') {
      throw new Error(
        'Actions may not have an undefined "type" property. ' +
          'Have you misspelled a constant?'
      )
    }
    //以上是对所传参数的一些限制

    //不能重复dispatch
    if (isDispatching) {
      throw new Error('Reducers may not dispatch actions.')
    }
    try {
      isDispatching = true  //初始为true
      // 每次发送 action，用于创建 store 的 `reducer` 都会被调用一// 次。调用时传入的参数是当前的状态以及被发送的 action。的返回// 值将被当作下一次的状态
      currentState = currentReducer(currentState, action)
    } finally {
      isDispatching = false
    }

    const listeners = (currentListeners = nextListeners) //更新监听器
    // 逐个执行监听器通知状态更改
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }
    //action参数原封不动返回
    return action
  }

  /**
   * 替换 store 当前使用的 reducer 函数。
   *
   * 如果你的程序代码实现了代码拆分，并且你希望动态加载某些 reducers。或
   * 者你为 redux 实现一个热加载的时候，你也会用到它。
   *
   * @param {Function} nextReducer 替换后的reducer
   * @returns {void}
   */
  function replaceReducer(nextReducer) {
    if (typeof nextReducer !== 'function') {
      throw new Error('Expected the nextReducer to be a function.')
    }

    currentReducer = nextReducer
    dispatch({ type: ActionTypes.REPLACE })
  }

  // observable方法，对subscribe的包装，方便搭配Rxjs使用
  function observable() {
    const outerSubscribe = subscribe
    return {
      subscribe(observer) {
        if (typeof observer !== 'object' || observer === null) {
          throw new TypeError('Expected the observer to be an object.')
        }

        function observeState() {
          if (observer.next) {
            observer.next(getState())
          }
        }

        observeState()
        const unsubscribe = outerSubscribe(observeState)
        return { unsubscribe }
      },

      [$$observable]() {
        return this
      }
    }
  }

  // 当一个store创建好，一个 "INIT" 的 action 就会分发，以便每个 reducer返回
  // 初始的状态,这有效填充初始的状态树。
  dispatch({ type: ActionTypes.INIT })

  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  }
}
```