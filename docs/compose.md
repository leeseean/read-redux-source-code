## 导读

我们经常使用compose来合并多个中间件middleware，那么compose内部是怎么合并多个middleware的呢？

下面上源码

## 源码

```
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```

如上所示，参数funcs是一组middleware函数，
如果没有传入middleware，则默认返回一个函数。
如果只传入一个middleware，则返回这个middleware。
如果传入多个middleware，运用js数组的原生reduce方法层层调用达到每个middleware都能调用到的效果。